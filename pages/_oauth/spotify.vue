<template>
  <v-app dark>
    <v-overlay dark>
      <v-row>
        <v-col class="text-center">
          <v-progress-circular indeterminate size="48" />
        </v-col>
      </v-row>
      <v-row>
        <v-col class="font-weight-light" v-if="state === null">
          Please wait, token is being processed.
        </v-col>
        <v-col class="font-weight-light" v-if="state === false">
           Unexpected error, please try to authenticate again.
        </v-col>
        <v-col class="font-weight-light" v-if="state === true">
          Saving token to a bot, refreshing back to a bot.
        </v-col>
      </v-row>
    </v-overlay>
  </v-app>
</template>

<script lang="ts">
import { getSocket } from '@sogebot/ui-helpers/socket';
import {
  defineComponent, Ref, ref,
} from '@vue/composition-api';

export default defineComponent({
  setup() {
    const socket = getSocket('/integrations/spotify');
    const state: Ref<boolean | null> = ref(null);

    if(window.location.hash || window.location.search) {
      socket.emit('spotify::state', (_err: string | null, spotifyState: any) => {
        let urlState = '';
        let urlCode = '';
        for (const url of window.location.search.split('&')) {
          if (url.startsWith('?code=') || url.startsWith('code=')) {
            urlCode = url.replace(/\??code=/, '');
          }
          if (url.startsWith('?state=') || url.startsWith('state=')) {
            urlState = url.replace(/\??state=/, '');
          }
        }

        if (urlState === spotifyState) {
          socket.emit('spotify::code', urlCode, () => {
            state.value = true;
            window.location.href = window.location.origin + '/#/settings/modules/integrations/spotify';
          });
        } else {
          state.value = false;
          console.error('State is not matching!');
        }
      });
    }

    return { state };
  },
});
</script>
