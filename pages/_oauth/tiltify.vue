<template>
  <v-app dark>
    <v-overlay dark>
      <v-row>
        <v-col class="text-center">
          <v-progress-circular indeterminate size="48" />
        </v-col>
      </v-row>
      <v-row>
        <v-col v-if="state === null" class="font-weight-light">
          Please wait, token is being processed.
        </v-col>
        <v-col v-if="state === false" class="font-weight-light">
          Unexpected error, please try to authenticate again.
        </v-col>
        <v-col v-if="state === true" class="font-weight-light">
          Saving token to a bot, refreshing back to a bot.
        </v-col>
      </v-row>
    </v-overlay>
  </v-app>
</template>

<script lang="ts">
import { getSocket } from '@sogebot/ui-helpers/socket'
import {
  defineComponent, Ref, ref
} from '@vue/composition-api'

export default defineComponent({
  setup () {
    const state: Ref<boolean | null> = ref(null)

    if (window.location.hash || window.location.search) {
      let urlCode = null;
      for (const url of window.location.search.split('&')) {
        if (url.startsWith('?token=') || url.startsWith('token=')) {
          urlCode = url.replace(/\??token=/, '')
        }
      }

      if (urlCode) {
        state.value = true
        getSocket('/integrations/tiltify').emit('tiltify::code', urlCode, () => {
          localStorage.tiltifySuccess = 'true';
          location.href = location.origin + '#/settings/modules/integrations/tiltify';
        });
      } else {
        state.value = false;
      }
    } else {
      location.href = `https://tiltify.soge.workers.dev/authorize?state=${window.location.origin}`
    }

    return { state }
  }
})
</script>
