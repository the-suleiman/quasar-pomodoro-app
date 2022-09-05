<template>
  <q-page class="flex flex-center">

    <timer-component/>

  </q-page>
</template>

<script>
import TimerComponent from "components/TimerComponent.vue";

export default {
  name: 'IndexPage',
  components: {TimerComponent}
}
</script>
