<template>
  <div id="app">
    <div id="content">
      <div id="game">
        <div 
          v-for="(cell, index) in cells" 
          :key="index"
          class="cell"
          :class="[cell, position == index + 1 ? 'position' : '']">
        </div>
      </div>

      <div class="answer">
        <button :class="[positionAnswer]">POSITION MATCH</button>
        <button :class="[audioAnswer]">AUDIO MATCH</button>
        <div>
          <button style="width: 100%">
            <input type="text" value="5">
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'app',

  data(){ return {
    /** render */
    cells: ["one", "two", "three", "four", "center", "five", "six", "seven", "eight"],
    position: null,
    letter: null,
    positionAnswer: '', // yes || no
    audioAnswer: '', // yes || no



    /** setup */
    // 
    keyStop: 'Escape',
    keyStart: 'Space',
    keyPosition: 'KeyA',
    keyAudio: 'KeyL',

    round: 30,
    playback: 1, // default
    letters: ['s', 't', 'l', 'q', 'o', 'e'],
    time: 2000, // miliseconds



    /** handle gameHistory */
    appHistory: {}, //{1: [97, 45, ...], ...}
    gameHistory: [], //[{},...]
    playPositions: [],
    playLetters: [],
    isPlaying: false,
    keyPositionPressed: false,
    keyListenerPressed: false,

    gamePositionResult: [],
    gameListenerResult: [],
    userPositionResult: [],
    userListenerResult: [],

    
    /** support */
    timeID: null,
    currentRound: 0
  }},

  computed: {
    backValue(){
      if (this.gameHistory.length >= this.playback) {
        let indexOf = this.gameHistory.length - 1 - this.playback
        return this.gameHistory[indexOf]
      }
      else {
        return false
      }
    }
  },

  methods: {
    start(){
      this.isPlaying = true
      this.playPositions = this.randomPositions(1, 9, true)
      this.playLetters = this.randomLetters(0, this.letters.length)
      this.timeID = setInterval(this.showAndPlaySound, this.time)
    },

    stop(){
      this.save()
    },

    save(){
      this.resetForNewGame()
      console.log('completed')
    },

    resetForNewGame(){
      this.isPlaying = false
      this.position = null
      this.keyPositionPressing = false
      this.keyPositionListener = false
      this.gameHistory.length = 0
      clearInterval(this.timeID)
    },

    resetForNextTime(){
      this.keyPositionPressing = false
      this.keyListenerPressing = false
      this.positionAnswer = ''
      this.audioAnswer = ''
    },

    showAndPlaySound(){
      // handle keyboard
      this.resetForNextTime()
      // show and speach
      this.position = this.playPositions.shift()
      this.letter = this.playLetters.shift()
      var synth = window.speechSynthesis;
      synth.speak(new SpeechSynthesisUtterance(this.letter));
      // save gameHistory
      this.gameHistory.push({position: this.position, letter: this.letter})
      console.log({position: this.position, letter: this.letter})
    },

    handleKeyPress(e){
      console.log(e)
      // play button
      if (e.code == this.keyStart && !this.isPlaying) { 
        this.isPlaying = true
        this.start()
      }

      //stop button
      if (e.code == this.keyStop && this.isPlaying) {
        this.isPlaying = false
        this.stop() 
      }

      // chỉ cho phép chơi khi backValue hợp lệ 
      if (this.isPlaying && this.backValue) {
        if (e.code == this.keyPosition && !this.keyPositionPressed) {
          this.keyPositionPressing = true
          this.positionAnswer = this.position == this.backValue.position ? "correct" : "fail"
        }

        if (e.code == this.keyAudio && !this.keyListenerPressed) {
          this.keyListenerPressing = true
          this.audioAnswer = this.letter == this.backValue.letter ? "correct" : "fail"
        }
      }
    },

    randomPositions(min, max, included){
      let tempPosiitons = []
      for (let i = 0; i< this.round; i++){
        let gameHistory = Math.floor(Math.random() * (max - min + included)) + min
        if (gameHistory == 5) { gameHistory += 1}
        tempPosiitons.push(gameHistory)
      }

      return tempPosiitons
    },

    randomLetters(min, max, included = false){
      let tempLetters = []
      for (let i = 0; i< this.round; i++){
        let gameHistory = Math.floor(Math.random() * (max - min + included)) + min
        tempLetters.push(this.letters[gameHistory])
      }

      return tempLetters
    },

    
  },

  watch: {
    gameHistory(){
      if (this.gameHistory.length > this.round) this.stop()
    },
  },

  // hooks
  mounted(){
    document.body.addEventListener('keydown', this.handleKeyPress)
  },

  destroyed(){
    if (this.isPlaying) {
      this.appHistory[this.playback] = 0
    }
  }
  
}
</script>

<style>
  * {
    margin: 0px;
    padding: 0px;
    font-size: 1.025em
  }
 
  #app #content {
    position: fixed;
    top: 50%;
    left: 50%;
    transform :translate(-50%, -50%);
  }

  #app #content > div {
    margin: 0 auto
  }

  

  #game{
    width: 300px;
    height: 300px;
    
    display: grid;
    grid-template-columns: repeat(3, 100px);
    grid-template-rows: repeat(3, 100px);
    grid-gap: 1px;
    background-color: gray
  }

  .cell {
    width: 100%;
    height: 100%;
    background-color: white;
    box-sizing: border-box;
    border: 10px solid white;
    transition: background-color 1s 1;
  }

  .one {
    grid-row: 1;
    grid-column: 1;
  }
  .two {
    grid-row: 1;
    grid-column: 2;
  }
  .three {
    grid-row: 1;
    grid-column: 3;
  }
  .four {
    grid-row: 2;
    grid-column: 1;
  }
  .five {
    grid-row: 2;
    grid-column: 3;
  }
  .six {
    grid-row: 3;
    grid-column: 1;
  }
  .seven {
    grid-row: 3;
    grid-column: 2;
  }
  .eight {
    grid-row: 3;
    grid-column: 3;
  }

  .cell.position {
    padding: 10px;
    background-clip: padding-box;
    background-color: dodgerblue
  }

  #app #content .answer {
    margin-top: 50px
  }
  .answer button {
    height: 50px;
    border-color: transparent;
    background-color: #f2f2f2;
    border-radius: 10px;
    font-weight: bolder;
    padding: 10px
  }

  .answer button:focus {
    outline: none
  }

  .answer .correct {
    background-color: #3e9841;
    color: white;
  }

  .answer .fail {
    background-color: #f44336;
    color: white;
  }

  
</style>
