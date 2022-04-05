<template>
  <v-app dark>
    <v-overlay dark>
      <v-row>
        <v-col class="text-center">
          <v-progress-circular indeterminate size="48" />
        </v-col>
      </v-row>
      <v-row>
        <v-col class="font-weight-light">
          Please wait until it is redirected back to bot.
        </v-col>
      </v-row>
    </v-overlay>
  </v-app>
</template>

<script lang="ts">
import { defineComponent, onMounted } from '@vue/composition-api'
import axios from 'axios'
import { get } from 'lodash'

export default defineComponent({
  setup () {
    onMounted(async () => {
      try {
        const url = new URL(window.location.href)
        const code = url.searchParams.get('code')
        const state = JSON.parse(window.atob(url.searchParams.get('state') ?? ''))

        // goto after login
        const gotoAfterLogin = sessionStorage.getItem('goto-after-login')
        sessionStorage.removeItem('goto-after-login')

        console.group('oauth::onMounted')
        console.debug({ url, code, state, gotoAfterLogin })
        console.groupEnd()

        // set it as new authorization
        localStorage.removeItem('accessToken')
        localStorage.removeItem('refreshToken')

        if (!code) {
          throw new Error('Missing code!')
        }

        const twitchHeaders = { headers: { Authorization: 'Bearer ' + code } }
        const twitchValidation = await axios.get<any>('https://id.twitch.tv/oauth2/validate', twitchHeaders)
        console.group('isUserLoggedIn::twitch::validation')
        console.debug({ twitchValidation, twitchHeaders })
        console.groupEnd()
        localStorage.setItem('userId', twitchValidation.data.user_id)
        localStorage.setItem('clientId', twitchValidation.data.client_id)

        await new Promise<void>((resolve, reject) => {
          const headers = {
            'x-twitch-token': code,
            'x-twitch-userid': twitchValidation.data.user_id
          }

          axios.get(`${process.env.isNuxtDev ? 'http://localhost:20000' : window.location.origin}/socket/validate`, {
            headers
          }).then(async (validation) => {
            console.group('isUserLoggedIn::bot::validation')
            console.debug({ validation, headers })
            console.groupEnd()
            localStorage.setItem('accessToken', validation.data.accessToken)
            localStorage.setItem('refreshToken', validation.data.refreshToken)
            localStorage.setItem('userType', validation.data.userType)

            const axiosData = await axios.get('https://api.twitch.tv/helix/users', {
              headers: {
                Authorization: 'Bearer ' + code,
                'Client-Id': twitchValidation.data.client_id
              }
            })
            const data = get(axiosData, 'data.data[0]', null)
            if (data === null) {
              localStorage.removeItem('userId')
              throw new Error('User must be logged')
            }
            localStorage.setItem('cached-logged-user', JSON.stringify(data))
            console.log(`Logged as ${twitchValidation.data.login}#${twitchValidation.data.user_id}`)
            resolve()
          }).catch(e => reject(e))
        })

        window.location.assign(`${gotoAfterLogin || state.referrer || state.url}`)
      } catch (error) {
        console.error({ error })
        window.location.assign(window.location.origin + '/credentials')
      }
    })
  }
})
</script>
