<template>
  <div id="app">
    <div id="menu">Brain Workshop</div>
    <div id="huongdan">
      <p><strong>Space bar</strong>: play</p>
      <p><strong>Escape</strong>: end game</p>
      <p><strong>A</strong>: position match</p>
      <p><strong>L</strong>: audio match</p>
    </div>

    <div id="setup">
      <p>n-back {{ playback }}<input type="range" min="1" max="6" step="1" v-model="playback" :disabled="isPlaying"></p>
      <p>speed {{ time }}<input type="range" min="500" max="3000" step="100" v-model="time" :disabled="isPlaying"></p>
    </div>

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
        <button :class="[positionAnswer]">
          POSITION MATCH 
          <div>
            <img src="./assets/like.png" alt=""> <span>{{positionMatchCount}}</span>
            <img src="./assets/dislike.png" class="dislike"> <span>{{positionFailCount}}</span>
          </div>
        </button>
        <button :class="[audioAnswer]">
          AUDIO MATCH
          <div>
            <img src="./assets/like.png" alt=""> <span>{{audioMatchCount}}</span>
            <img src="./assets/dislike.png" class="dislike"> <span>{{audioFailCount}}</span>
          </div>
        </button>
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



    /** setup */
    // 
    keyStop: 'Escape',
    keyStart: 'Space',
    keyPosition: 'KeyA',
    keyAudio: 'KeyL',
    playback: 1, // default
    letters: ['s', 't', 'l', 'q'],
    time: 2000, // miliseconds



    /** handle gameHistory */
    appHistory: {}, //{1: [97, 45, ...], ...}
    gameHistory: [], //[{},...]
    playPositions: [],
    playLetters: [],
    isPlaying: false,
    keyPositionPressed: false,
    keyListenerPressed: false,

    userPositionResult: [],
    userListenerResult: [],

    position: null,
    letter: null,
    positionAnswer: '', // yes || no
    audioAnswer: '', // yes || no

    
    /** support */
    timeID: null,
    currentRound: 0
  }},

  computed: {
    round(){
      return 20 + this.playback
    },
      
    backValue(){
      if (this.gameHistory.length >= this.playback) {
        let indexOf = this.gameHistory.length - 1 - this.playback
        return this.gameHistory[indexOf]
      }
      else {
        return false
      }
    },

    positionMatched(){
      return this.backValue && this.position == this.backValue.position
    },
    audioMatched(){
      return this.backValue && this.letter == this.backValue.letter
    },

    positionMatchCount(){
      return this.userPositionResult.filter( result => result == true).length
    },
    positionFailCount(){
      return this.userPositionResult.filter( result => result == false).length
    },
    audioMatchCount(){
      return this.userListenerResult.filter( result => result == true).length
    },
    audioFailCount(){
      return this.userListenerResult.filter( result => result == false).length
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
      this.clearCurrentRound()
      this.clearCurrentGame()
      clearInterval(this.timeID)
    },

    resetForNextTime(){
      if (this.positionMatched) { this.userPositionResult.push(this.keyPositionPressed) }
      else if (this.keyPositionPressed) { this.userPositionResult.push(this.positionMatched) }

      if (this.audioMatched) { this.userListenerResult.push(this.keyListenerPressed) }
      else if (this.keyListenerPressed) { this.userListenerResult.push(this.audioMatched) }
      
      this.clearCurrentRound()
    },

    clearCurrentRound(){
      this.keyPositionPressed = false
      this.keyListenerPressed = false
      this.positionAnswer = ''
      this.audioAnswer = ''
      this.position = null
      this.letter = null
    },

    clearCurrentGame(){
      this.isPlaying = false
      this.userPositionResult = []
      this.userListenerResult = []
      this.gameHistory = []
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
          this.keyPositionPressed = true
          this.positionAnswer = this.position == this.backValue.position ? "correct" : "fail"
        }

        if (e.code == this.keyAudio && !this.keyListenerPressed) {
          this.keyListenerPressed = true
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

  #menu {
    background-color:dodgerblue;
    color: white;
    height: 56px;
    line-height: 56px;
    font-weight: bolder;
    font-size: 30px;
    padding: 10px;
    box-shadow: 0px 1px 3px 1px grey;
    margin-bottom: 20px; 
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
    min-height: 50px;
    border-color: transparent;
    background-color: #f2f2f2;
    border-radius: 10px;
    font-weight: bolder;
    padding: 10px
  }

  .answer button:focus {
    outline: none
  }

  button img {
    width: 30px;
    height: 30px;
  }

  img.dislike {
    transform: scaleY(-1)
  }

  button * {
    vertical-align: middle
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
