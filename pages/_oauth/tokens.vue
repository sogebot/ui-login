<template>
  <v-app dark>
    <v-overlay dark>
      <v-row v-if="!accessToken || !refreshToken || !_clientId ">
        <v-col class="text-center">
          <v-progress-circular indeterminate size="48" />
        </v-col>
      </v-row>
      <v-row v-else>
        <v-col>
          <pre>
Access token: {{ accessToken }}
Refresh token: {{ refreshToken }}
Client ID: {{ _clientId }}
          </pre>
        </v-col>
      </v-row>
    </v-overlay>
  </v-app>
</template>

<script lang="ts">
import { defineComponent, useRoute, onMounted, ref } from '@nuxtjs/composition-api'

export default defineComponent({
  setup () {
    const route = useRoute()

    const accessToken = ref(null as null | string)
    const refreshToken = ref(null as null | string)
    const _clientId = ref(null as null | string)

    onMounted(async () => {
      const { scope, clientId, code, clientSecret, state } = route.value.query
      const redirectUri = `${location.origin}${location.pathname}`

      const stateIn = btoa(JSON.stringify({ clientSecret, clientId }))

      if (!code) {
        location.href = `https://id.twitch.tv/oauth2/authorize?client_id=${clientId}&redirect_uri=${redirectUri}&response_type=code&scope=${scope}&force_verify=true&state=${stateIn}`
      } else {
        try {
          // get clientSecret from state
          const client = JSON.parse(atob(String(state)))
          const response = await fetch(`https://id.twitch.tv/oauth2/token?client_id=${client.clientId}&client_secret=${client.clientSecret}&code=${code}&grant_type=authorization_code&redirect_uri=${redirectUri}`, {
            method: 'POST'
          })
          if (response.ok) {
            const data = await response.json()
            console.log({ data })
            accessToken.value = data.access_token
            refreshToken.value = data.refresh_token
            _clientId.value = client.clientId
          }
        } catch (e) {
          console.error(e)
        }
      }
    })

    return {
      accessToken, refreshToken, _clientId
    }
  }
})
</script>
