<script setup>
import { ref, onMounted } from 'vue'
import Papa from 'papaparse'

const headlinetext = ref('')
const text1 = ref('')
const text2 = ref('')
const nextStepHeadlinetext = ref('')
const remainingTimeDisplay = ref('')
const stepnumberDisplay = ref('')
const totalTimeDisplay = ref('')

var nextStepTime = 0
var intervalIndex
var stepnumber = 0
var totalStartTime = 0


var scheduleData = []

const formatTime = seconds => {
  const minutes = Math.floor(seconds / 60)
  const remainingSeconds = seconds % 60;
  return `${String(minutes).padStart(2, '0')}:${String(remainingSeconds).padStart(2, '0')}`
}

const setCurrentStepInfo = () => {
  headlinetext.value = scheduleData[stepnumber].headlinetext
  if(stepnumber + 1 >= scheduleData.length)
    nextStepHeadlinetext.value = ''
  else
    nextStepHeadlinetext.value = scheduleData[stepnumber + 1].headlinetext // Checken, ob es einen naechsten Step gibt!
  text1.value = scheduleData[stepnumber].text1
  text2.value = scheduleData[stepnumber].text2  
  stepnumberDisplay.value = stepnumber + 1
  if (scheduleData[stepnumber].time == 0) {
    nextStepTime = 0
    remainingTimeDisplay.value = '-->'
  }
  else {
    nextStepTime = Date.now() + scheduleData[stepnumber].time * 1000
    remainingTimeDisplay.value = formatTime(scheduleData[stepnumber].time)
  }
}

const notify = () => {
  totalTimeDisplay.value = formatTime(Math.floor((Date.now() - totalStartTime) / 1000))
  let remainingTime = nextStepTime - Date.now()
  if(nextStepTime > 0 && remainingTime < 0) {
    document.getElementById("bleep").play()
    if(stepnumber.value == scheduleData.length + 1) {
      clearInterval(intervalIndex)
      headlinetext.value = 'Fertsch'
      stepnumber.value = ''
      text1.value = ''
      text2.value = ''
      remainingTimeDisplay.value = ''
    }
    else {
      stepnumber++
      setCurrentStepInfo()
    }
  }
  else {
    if(nextStepTime > 0)
      remainingTimeDisplay.value = formatTime(Math.floor(remainingTime / 1000) + 1)
  }
}

const jumpToPreviousStep = () => {
  stepnumber--;
  if(stepnumber < 0) stepnumber = 0
  setCurrentStepInfo()
}

const jumpToNextStep = () => {
  stepnumber++; // TODO check if there's anything!
  setCurrentStepInfo()
}

onMounted(() => {
  fetch('/download/fit/schedule.csv')
    .then(response => response.text())
    .then(text => {
      const data = Papa.parse(text, { header: false })
      for(const row of data.data) {
	if(row.length != 4) {
	  console.log('CSV file error in ' + row[0])
	  return
	}
      }
      scheduleData = data.data.map(row => ({ headlinetext: row[0], text1: row[1], text2: row[2], time: row[3]}))
      setCurrentStepInfo()
      totalStartTime = Date.now()
      intervalIndex = setInterval(() => { notify() }, 200)
    })
})

</script>
				    
<template>
  <audio id="bleep" src="bleep.mp3"></audio>				      
 
  <h3>{{ stepnumberDisplay }}. {{ headlinetext }}</h3>
  <h4>-> {{ nextStepHeadlinetext }}</h4>
  <div class="textdisplay">{{ text1 }}</div>
  <div class="textdisplay">{{ text2 }}</div>
  <div class="foot-box">
  <div class="remaining-time">{{ remainingTimeDisplay }}</div>
  <div>
  <button>&vert;&vert;</button>
  <button @click="jumpToPreviousStep()">&vert;&lt;</button>
  <button @click="jumpToNextStep()">&gt;&vert;</button>
  </div>
 <div class="total-time">{{ totalTimeDisplay }}</div>
  </div>
</template>


<style scoped>



</style>
