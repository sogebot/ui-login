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

export default defineComponent({
  setup () {
    onMounted(() => {
      try {
        const url = new URL(window.location.href)
        const code = url.searchParams.get('code')
        const state = JSON.parse(window.atob(url.searchParams.get('state') ?? ''))

        // save token code to localStorage
        localStorage.setItem('code', code ?? '')

        // set it as new authorization
        localStorage.removeItem('accessToken')
        localStorage.removeItem('refreshToken')

        // goto after login
        const gotoAfterLogin = sessionStorage.getItem('goto-after-login')
        sessionStorage.removeItem('goto-after-login')

        window.location.assign(`${gotoAfterLogin || state.referrer || state.url}`)
      } catch (error) {
        console.log({ error })
        window.location.assign(window.location.origin + '/login')
      }
    })
  }
})
</script>
