<template>
  <div id="app">
    <br/> <br/> <br/> 
    <time-div-select v-model="timeSeries"></time-div-select>
  </div>
</template>

<script>
import timeDivSelect from './components/timeDivSelect.vue'
let _timeArray = []
let _timeString = ''
for (let i = 0; i < 48 * 7; i++) {
  _timeArray.push('0')
}
_timeString = _timeArray.join('')
export default {
  name: 'app',
  data(){
    return {
      timeSeries:_timeString
    }
  },
  components: {
    timeDivSelect
  },
  watch:{
    timeSeries(val){
      console.log(val)

    }
  }
}
</script>

<style>
</style>
