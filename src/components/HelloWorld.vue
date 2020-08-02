<template>
  <div>
    <img class="parent" src="../assets/resources/bg_30FRM_loop.gif" />
    
      <img v-if="expression === undefined || expression === null" class="child" src="../assets/resources/idle/MAIN_FRAME.gif" />

      <img v-if="expression === 'idle1'" class="child" src="../assets/resources/idle/IDLE_15_FRMS_V2.gif" />
      <img v-if="expression === 'idle2'" class="child" src="../assets/resources/idle/IDLE_25_FRMS_V3.gif" />
      <img v-if="expression === 'idle3'" class="child" src="../assets/resources/idle/IDLE_15_FRMS_V1.gif" />
   
  </div>
</template>
<script>
export default {
  name: 'HelloWorld',
  data: () => ({
    recognition: null,
    talking: false,
    expression: null,
    list: ['undefined', 'idle1', 'idle2', 'idle3']
  }),
  methods: {
    timer() {
      setInterval(() => {
        // make switch for transition expression from normal to angry or kind
        // make if statement to determine what list to use when transitioned
          this.expression = this.list[this.random()];
          setTimeout(() => {
              this.expression = null;
          }, 5000)
      }, 12000)
    },
    random() {
      return Math.floor(Math.random() * Math.floor(this.list.length)) + 1;
    },
    face(face) {
      switch(face) {
        case 'angry':
        break;
        case 'kind':
        break;
        case 'normal':
        break;
      }
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
    detectSilence(
      stream,
    onSoundEnd = {},
    onSoundStart = {},
    silence_delay = 500,
    min_decibels = -80) {
      const ctx = new AudioContext();
      const analyser = ctx.createAnalyser();
      const streamNode = ctx.createMediaStreamSource(stream);
      streamNode.connect(analyser);
      analyser.minDecibels = min_decibels;

      const data = new Uint8Array(analyser.frequencyBinCount); // will hold our data
      let silence_start = performance.now();
      let triggered = false; // trigger only once per silence event

      function loop(time) {
        requestAnimationFrame(loop); // we'll loop every 60th of a second to check
        analyser.getByteFrequencyData(data); // get current data
        if (data.some(v => v)) { // if there is data above the given db limit
          if(triggered){
            triggered = false;
            onSoundStart();
            }
          silence_start = time; // set it to now
        }
        if (!triggered && time - silence_start > silence_delay) {
          onSoundEnd();
          triggered = true;
        }
      }
      loop();
    }
  },
  created() {
    navigator.mediaDevices.getUserMedia({audio:true}).then(stream => this.detectSilence(stream, this.stopSpeech, this.startSpeech))
    .catch(e => console.log(e.message));
  },
  mounted() {
    this.startup();
    this.timer();
  }
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

