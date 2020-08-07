<template>
  <div>
    <img class="parent" src="../assets/resources/bg_30FRM_loop.gif" />
    
      <img v-if="expression === undefined || expression === null && mood === null" class="child" src="../assets/resources/idle/MAIN_FRAME.gif" />

      <img v-if="expression === 'idle1'" class="child" src="../assets/resources/angry/IDLE_15_FRMS_V2.gif" />
      <img v-if="expression === 'idle2'" class="child" src="../assets/resources/angry/IDLE_25_FRMS_V3.gif" />
      <img v-if="expression === 'idle3'" class="child" src="../assets/resources/angry/IDLE_15_FRMS_V1.gif" />

      <img v-if="expression === undefined || expression === null && mood === 'angry'" class="child" src="../assets/resources/angry/MAIN_FRAME.gif" />

      <img v-if="expression === null && this.talking === true" class="child" src="../assets/resources/angry/talk/TALKING_10_FRMS_V1.gif" />
   
  </div>
</template>
<script>
export default {
  name: 'HelloWorld',
  data: () => ({
    recognition: null,
    expression: null,
    mood: 'angry',
    talking: false,
    idle: ['undefined', 'idle1', 'idle2', 'idle3'],
    angry: [],
    transition: ['atn', 'nta']
  }),
  methods: {
    timer() {
      setInterval(() => {
        // make switch for transition expression from normal to angry or kind
        // make if statement to determine what list to use when transitioned
        if(!this.talking) {
          this.expression = this.idle[this.random()];
          setTimeout(() => {
              this.expression = null;
          }, 5000)
        }
      }, 12000)
      
    },
    random() {
      return Math.floor(Math.random() * Math.floor(this.idle.length)) + 1;
    },
    randomtalk() {
      return Math.random() < 0.5 ? 'v1' : 'v2'
    },
    startup() {
      const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
      if (!SpeechRecognition && "development" !== 'production') {
        throw new Error('Speech Recognition does not exist on this browser. Use Chrome or Firefox');
      }
      if (!SpeechRecognition) {
        console.log("No Speech Recognition");
        return;
      }
      console.log("Starting UP");
      this.recognition = SpeechRecognition? new SpeechRecognition() : false;

      this.recognition.lang = 'en-US'
      this.recognition.continuous = true
      this.recognition.interimResults = false;

    },
    stopSpeech(){
      this.recognition.stop();
    },
    startSpeech(){
      try{ 
        this.recognition.start();
    }
      catch(e){
        console.log(e)
      }
    },
    activeTalk() {
      this.talking = true;
    },
    deativeTalk() {
      this.talking = false;
    },
    detectSilence(
      stream,
    onSoundEnd = {},
    onSoundStart = {},
    onTalkStart = {},
    onTalkend = {},
    silence_delay = 500,
    min_decibels = -80) {
      const ctx = new AudioContext();
      const analyser = ctx.createAnalyser();
      const streamNode = ctx.createMediaStreamSource(stream);
      let bufferLen = analyser.frequencyBinCount;
      streamNode.connect(analyser);
      analyser.minDecibels = min_decibels;
      analyser.fftSize = 256;
     
      const data = new Uint8Array(bufferLen); // will hold our data
      
      
     

      let silence_start = performance.now();
      let triggered = false; // trigger only once per silence event

      function loop(time) {
        requestAnimationFrame(loop); // we'll loop every 60th of a second to check
        analyser.getByteFrequencyData(data); // get current data
        // console.log(data)
        if (data.some(v => v)) { // if there is data above the given db limit
          //console.log('start')
          onTalkStart();

          if(triggered){
            triggered = false;
            onSoundStart();
            }
          silence_start = time; // set it to now
        }
        if (!triggered && time - silence_start > silence_delay) {
          //console.log('end')
          onTalkend();
          onSoundEnd();
          
          triggered = true;
        }
      }
      loop();
    }
  },
  created() {
    navigator.mediaDevices.getUserMedia({audio:true}).then(stream => this.detectSilence(stream, this.stopSpeech, this.startSpeech, this.activeTalk, this.deativeTalk))
    .catch(e => console.log(e.message));
  },
  mounted() {
    this.startup();
    this.timer();
  },
}
</script>
<style scoped>
  .parent {
    position: relative;
    top: 0;
    left: 0;
  }
  .child {
    position: absolute;
    top: 9x;
    left: 0;
  }
</style>

