<template>
  <div class="q-col">
    <div class="text-center">
      <h3 style="margin: 0;">{{ actualStep.label }}</h3>
      <h4>{{ actualRound }}/{{ rounds }}</h4>
    </div>


    <div class="text-center">

      <q-circular-progress
        show-value
        font-size="16px"
        class="q-ma-md"
        :value="timerValue"
        size="300px"
        :thickness="0.02"
        color="red"
        track-color="grey-3"
        reverse
      >
        <h2>{{ timerDisplay }}</h2>
      </q-circular-progress>

      <div class="q-pa-md q-gutter-sm">
        <q-btn icon="play_arrow" size="xl" @click="initCountdownTimer"/>
        <q-btn icon="pause" size="xl" @click="stopCountdownTimer"/>
      </div>

      <div class="q-pa-md q-gutter-sm">
        <q-btn label="Settings" color="primary" @click="settingsDialog = true"/>
        <q-dialog v-model="settingsDialog" persistent>
          <q-card style="min-width: 350px">
            <q-card-section>
              <div class="text-h6">Focus time</div>
            </q-card-section>

            <q-card-section class="q-pt-none">
              <q-input dense v-model="settings.focus" mask="time"/>
            </q-card-section>

            <q-card-section>
              <div class="text-h6">Short break</div>
            </q-card-section>

            <q-card-section class="q-pt-none">
              <q-input dense v-model="settings.shortBreak" mask="time"/>
            </q-card-section>

            <q-card-section>
              <div class="text-h6">Long break</div>
            </q-card-section>

            <q-card-section class="q-pt-none">
              <q-input dense v-model="settings.longBreak" mask="time"/>
            </q-card-section>


            <q-card-section>
              <div class="text-h6">Rounds</div>
            </q-card-section>

            <q-card-section class="q-pt-none">
              <q-input dense v-model="settings.rounds" type="number"/>
            </q-card-section>

            <q-card-actions align="right">
              <q-btn flat label="Cancel" v-close-popup/>
              <q-btn flat label="Apply settings" v-close-popup @click="applySettings(settings)"/>
            </q-card-actions>
          </q-card>
        </q-dialog>
      </div>

      <div class="q-pa-md q-gutter-sm">
        <q-btn label="Focus" @click="setActualStep(this.steps.focus)"/>
        <q-btn label="Short break" @click="setActualStep(this.steps.shortBreak)"/>
        <q-btn label="Long break" @click="setActualStep(this.steps.longBreak)"/>
      </div>

    </div>
  </div>
</template>

<script>
import {ref, watch} from 'vue'
import moment from 'moment'
import timer from '/src/enums/timer'
import alarm from '/src/sounds/alarm.mp3'

const {FOCUS, SHORT, LONG, ROUNDS} = timer

export default {
  name: "TimerComponent",
  setup() {

    let steps = ref({
      focus: {
        time: FOCUS.TIME,
        label: FOCUS.LABEL,
        seconds: FOCUS.SECONDS
      },
      shortBreak: {
        time: SHORT.TIME,
        label: SHORT.LABEL,
        seconds: SHORT.SECONDS
      },
      longBreak: {
        time: LONG.TIME,
        label: LONG.LABEL,
        seconds: LONG.SECONDS
      }
    })

    let settings = ref({
      focus: FOCUS.TIME,
      shortBreak: SHORT.TIME,
      longBreak: LONG.TIME,
      rounds: ROUNDS
    })

    const timerValue = ref(100)

    let actualStep = ref(null)

    let selectedTime = ref(null)

    let timerDisplay = ref('')

    let timerInterval = ref(null)

    const actualRound = ref(1)

    const settingsDialog = ref(false)

    let rounds = ref(ROUNDS)

    setActualStep(steps.value.focus)

    function setActualStep(value) {
      actualStep.value = value
      setTimer(actualStep.value.seconds)
    }

    function setActualRound(value) {
      actualRound.value = value
    }

    function setNextStep() {
      const {label} = actualStep
      if (label === LONG.LABEL) {
        playAlarm()
        setActualRound(1)
        setActualStep(steps.value.focus)
        stopCountdownTimer()
      } else if (label === FOCUS.LABEL && actualRound.value === rounds) {
        playAlarm()
        setActualStep(steps.value.longBreak)
      } else if (label === FOCUS.LABEL && actualRound.value !== rounds) {
        playAlarm()
        setActualStep(steps.value.shortBreak)
      } else if (label === SHORT.LABEL && actualRound.value !== rounds) {
        playAlarm()
        setActualStep(steps.value.focus)
        setActualRound(actualRound.value + 1)
      }
    }

    function timerPercentage() {
      timerValue.value = (+timerDisplay.value.split(':')[0] * 60 + +timerDisplay.value.split(':')[1]) / selectedTime.value * 100
    }

    function setTimer(value) {
      timerDisplayConvert(value)
      selectedTime.value = value
    }

    function timerDisplayConvert(secondsLeft) {
      const minutes = Math.floor((secondsLeft % 3600) / 60)
      const seconds = secondsLeft % 60

      function addZero(value) {
        return value < 10 ? `0${value}` : value
      }

      timerDisplay.value = `${addZero(minutes)}:${addZero(seconds)}`
    }

    function subtractOneSecond() {
      timerDisplay.value = moment(timerDisplay.value, 'mm:ss').subtract(1, 'seconds').format('mm:ss')
    }

    function initCountdownTimer() {
      timerInterval = setInterval(() => {
        subtractOneSecond()
        timerPercentage()
      }, 1000)
    }

    function stopCountdownTimer() {
      clearInterval(timerInterval)
    }

    function applySettings(settings) {
      Object.entries(settings).forEach(([key, value]) => {
        if (key === 'rounds') {
          rounds = +value
        } else {
          steps.value[key].time = value
          steps.value[key].seconds = +value.split(':')[0] * 60 + +value.split(':')[1]
        }
        setActualRound(1)
        setActualStep(steps.value.focus)
      })
    }

    function playAlarm() {
      const audio = new Audio(alarm)
      audio.play()
    }

    watch(() => timerDisplay.value, (value, oldValue) => {
      if (oldValue === '00:02' && value === '00:01')
        setTimeout(() => {
          setNextStep()
        }, 1000)
    })

    return {
      timerValue,
      timerDisplay,
      initCountdownTimer,
      stopCountdownTimer,
      setActualStep,
      actualStep,
      actualRound,
      rounds,
      steps,
      settingsDialog,
      settings,
      applySettings
    }
  }
}
</script>

<style scoped>
* {
  font-weight: 200;
}
</style>
