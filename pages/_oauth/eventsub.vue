<template>
  <v-app dark>
    <v-overlay dark>
      <v-row>
        <v-col class="text-center green--text">
          <v-icon class="green--text" size="12rem">
            {{ mdiCheckboxMarkedCircleOutline }}
          </v-icon>
          <p>
            Login successful. Please close this window.
          </p>
        </v-col>
      </v-row>
    </v-overlay>
  </v-app>
</template>

<script lang="ts">
import { defineComponent } from '@vue/composition-api'
import { mdiCheckboxMarkedCircleOutline } from '@mdi/js'

export default defineComponent({
  setup () {
    return { mdiCheckboxMarkedCircleOutline }
  }
})
</script>
