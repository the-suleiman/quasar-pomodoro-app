<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-icon name="av_timer" size="30px"/>

        <q-toolbar-title>
          Pomodoro Timer
        </q-toolbar-title>

        <div>
          <q-toggle
            v-model="darkMode"
            checked-icon="brightness_2"
            unchecked-icon="wb_sunny"
            :model-value="darkMode"/>
        </div>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import {ref, watch} from 'vue'
import {useQuasar} from "quasar";

export default{
  name: 'MainLayout',

  setup() {
    const $q = useQuasar()

    $q.dark.set(true)

    const darkMode = ref(true)

    watch(darkMode, val => {
      $q.dark.set(val)
    })


    return {
      darkMode,
    }
  }
}
</script>
