<template>
  <div class="index">
    <div>
      <!-- <Canvas :canvas-id="'canvas-one'" ref="childCanvas"/> -->
    </div>
    <div class="screen">
      <ul class="nav-group">
        <li>
          <ul class="nav">
            <li class="nav-item nav-item-selected">games</li>
            <li class="nav-item disabled">tutorials</li>
            <li class="nav-item disabled">social</li>
          </ul>
        </li>
        <div>
          <div class="profile-card"></div>
          <p>Ryan's Portfolio</p>
        </div>
      </ul>
      <div class="game-count">{{(games && selectedGameIndex >= 0)? `${selectedGameIndex + 1}/${games.length}` :""}}</div>
      <div class="game-list">
        <ul class="icon-tray" :style="{left: (gameBarXOffset + (gameBarX * -1)) + 'rem'}">
          <li v-for="(item, index) in gameList" :key="index" :class="{'icon':true, 'icon-selected' : index === selectedGameIndex}" @click="selectGame(index)" @mouseenter="hoverTitleIndex = index" @mouseleave="hoverTitleIndex = null">
            <div :class="{'icon-cicrle' : index === selectedGameIndex}"></div>
            <div class="icon-img" v-bind:style="{ backgroundImage : `url(${item.icon}`}"></div>
            <div :class="{ 'highlight-title': index === hoverTitleIndex}">{{(item.title.length > 12)? item.title.substring(0,15) + "..." : item.title}}</div>
          </li>
        </ul>
        <div class="horizontal-scroll scroll-left" @mouseenter="moveDirection = 'left'" @dragenter="moveDirection = 'left'" @mouseleave="moveDirection = null"  @dragleave="moveDirection = null"></div>
        <div class="horizontal-scroll scroll-right" @mouseenter="moveDirection = 'right'" @dragenter="moveDirection = 'right'" @mouseleave="moveDirection = null"  @dragleave="moveDirection = null"></div>
      </div>
      <div v-if="selectedGame !== null">
        <div class="game-group">
          <div class="game-group-details">
            <img class="title" :src="titleImage" :alt="selectedGame.title">
            <div class="game-group-details-controls">
                <a class="pill pill-big blue" :style="{display: (selectedGame.play)? 'inherit' : 'none'}" target="_blank" :href="selectedGame.play">Play/Download</a>
                <a class="pill" :style="{display: (selectedGame.source)? 'inherit' : 'none'}" target="_blank" :href="selectedGame.source">Source Code</a>
                <a class="pill" :style="{display: (selectedGame.demo)? 'inherit' : 'none'}">Video</a>
            </div>
          </div>
          <img class="game-group-poster" :src="posterImage"/>
        </div>
        <div class="description">
          <h3>{{selectedGame.subtitle}}</h3>
          <article v-html="selectedGame.description"></article>
          <div class="description-additional">
            <div>Title: {{selectedGame.title || '-'}}</div>
            <div>Creation date: {{selectedGame.date || '-'}}</div>
            <div>Tags:
              <div class="tag" v-for="(tag) in selectedGame.tags" :key="tag">
                {{tag}}
              </div>
              <div class="empty" v-if="!selectedGame.tags">
                -
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Canvas from "../components/Canvas";
export default {
  data () {
    return {
      games: [],
      selectedGameIndex: null,
      hoverTitleIndex: null,
      selectedGame: null,
      gameBarX: 0,
      gameBarXLimit: 0,
      gameBarXOffset: 0.8,
      gameBarXSpeed: 14,
      moveDirection: null,
      lastUpdate: null,
      path: null,
      scope: null
    }
  },
  async fetch() {
    const data = await import(`@/static/data.json`)

    this.games = []
    if(data !== null)
    {
      data.games.forEach((game) => {
        this.games.push(game)
        this.selectGame(0);
      })
    }
  },
  fetchOnServer: false,
  computed: {
    gameList () {
      const gameSelectionArray = []

      if (this.games !== null) {
        this.games.forEach((game) => {

          let icon = null

          try {
            icon = require(`~/assets/${game.project}-icon.png`)
          } catch (err) {
            icon = require(`~/assets/question-icon.png`)
          }

          gameSelectionArray.push({
            icon: icon,
            title: game.title
          })

          this.gameBarXLimit += 4.2;
        })
      }
      return gameSelectionArray
    },
    posterImage () {
      let image = null
      
      try { 
        image = require(`~/assets/${this.selectedGame.project}-full.png`)
      } catch (err) {
        image = null
      }

      return image
    },
    titleImage () {
      let image = null
      
      try { 
        image = require(`~/assets/${this.selectedGame.project}-title.png`)
      } catch (err) {
        image = null
      }

      return image
    }
  },
  methods: {
    selectGame(index) {
      this.selectedGameIndex = index
      this.selectedGame = this.games[index]
    },
    scrollRight(dt) {
      this.gameBarX += this.gameBarXSpeed * dt

      if(this.gameBarX > this.gameBarXLimit) {
         this.gameBarX = this.gameBarXLimit
      }
    },
    scrollLeft(dt) {
      this.gameBarX -= this.gameBarXSpeed * dt

      if(this.gameBarX < 0) {
        this.gameBarX = 0
      }
    },
    tick() {
        var now = Date.now()

        if(this.lastUpdate === null) {
          this.lastUpdate = now
        }

        var dt = now - this.lastUpdate
        dt /= 1000
        this.lastUpdate = now

        switch(this.moveDirection) {
          case 'left':
            this.scrollLeft(dt)
            break;
          case 'right':
            this.scrollRight(dt)
            break;
        }
    },
    reset() {
      this.$refs.childCanvas.reset();
    }
  },
  components: {
      Canvas
  },
  mounted () {
    this.lastUpdate = Date.now();
    var myInterval = setInterval(this.tick, 0);
  }
}
</script>

<style>
html,body{
  margin: 0;
  padding: 0;
  color: white;
  max-width: 100vw;
  overflow-x: hidden;
  background: #1e1e2a;
}

.index {
  margin: 0 auto;
  min-height: 100vh;
  overflow: hidden;
  z-index: 2;
}

.screen {
  margin: 0 auto;
  min-height: 100vh;
  max-width: 60rem;
  overflow: hidden;
  z-index: 2;
}

.screen > * {
  position: relative;
}

div {
  z-index: 3;
}

.disabled {
  text-decoration: line-through !important;
}

ul, li {
  list-style: none;
}

.nav-group {
  display: flex;
  font-size: 2rem;
  flex-direction: row;
  flex-wrap: wrap-reverse;
  justify-content: space-around;
  margin: 0;
  padding: 0;
}

.nav {
  display: flex;
  font-size: 2rem;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: stretch;
  margin: 0;
  padding: 0;
}

.nav-item {
  margin: 1rem;
  opacity: 0.5;

  transition: all 0.5s;
}

.nav-item:hover {
  opacity: 1;
  text-decoration: underline;
}

.nav-item-selected {
  opacity: 1;
}

.profile-card {
  position: relative;
  width: 6rem;
  height: 6rem;
  border-radius: .2rem;
  border: .2rem solid #fff;
  top: 1rem;
  margin: 0 auto;
  background-image: url('~@/assets/profile.jpg');
  background-repeat: no-repeat;
  background-size: cover;
}

.horizontal-scroll{
  width: 3rem;
  height: 4.8rem;
  display: flex;
  list-style: none;
  opacity: 0.8;
  font-size: 3rem;
  justify-content: center;
  margin: 0 auto;
  color: #fff;
  position: absolute;
  transition: opacity 0.5s;
  background: linear-gradient(0deg, rgb(0, 0, 0, 0) 0,  rgb(0, 0, 0, 0.5) 25%, rgb(0, 0, 0, 0.5) 75%, rgba(0, 0, 0, 0.0) 100%);
}

.horizontal-scroll:hover{
  opacity: 1;
  /* background: linear-gradient(0deg, rgb(0, 0, 0, 0) 0,  rgb(0, 0, 0, 0.5) 25%, rgb(0, 0, 0, 0.5) 45%, rgba(255, 255, 255, 1) 50%, rgb(0, 0, 0, 0.5) 55%, rgb(0, 0, 0, 0.5) 75%, rgba(0, 0, 0, 1) 100%); */
}

.scroll-left{
  left: 0;
}

.scroll-left::after{
  content: "<";
}

.scroll-right{
  right: 0;
}

.scroll-right::after{
  content: ">";
}

.icon-tray{
  height: 4.8rem;
  display: flex;
  position: absolute;
}

.icon {
  transition: 0.2s;
  width: 4rem;
  height: 4rem;
  border-radius: 0.4rem;
  margin: 0 0.4rem;
  overflow: hidden;
}

.icon:hover {
  width: 6rem;
  height: 6rem;
  padding: 0.2rem;
  background: #1e6eff;
}

.icon-selected{
  width: 6rem;
  height: 6rem;
  padding: 0.2rem;
}

@keyframes icon-animation {
  from {
    transform:  scale(1.5,1.5) rotate(0deg);
  }
  to {
    transform:  scale(1.5,1.5) rotate(360deg);
  }
}

.icon-cicrle {
  position: static;
  width: 6rem;
  height: 6rem;
  background: linear-gradient(0deg, rgb(255, 86, 86, 0.7) 0px, rgba(30, 120, 255, 0.7) 100%);
  animation-name: icon-animation;
  animation-duration: 2s;
  animation-iteration-count: infinite;
  animation-timing-function: linear;
  transform: scale(2,2);
}

.icon-img {
  background: #000;
  width: 100%;
  height: 100%;
  margin: auto;
  border-radius: 0.3rem;
  background: #000;
  background-image: url('~@/assets/question-icon.png');
  background-repeat: no-repeat;
  background-size: contain;
  background-position: center;
  border: solid 0.1rem black;
  box-sizing: border-box;
  position: sticky;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
}

.highlight-title {
  position: absolute;
  top: 6rem;
  width: 20rem;
}

.game-count {
  position: relative;
  /* top: 2rem; */
  display: block;
  text-align: right;
  margin: 2rem 2rem 0 0;
  font-weight: 100;
  opacity: 0.5;
}

.game-list {
  position: relative;
  /* top: 2rem; */
  left: 0;
  right: 0;
  height: 10rem;
}

.game-group {
  display: flex;
  flex-flow: row;
  flex-wrap: wrap-reverse;
  margin: 0.2rem 2rem;
  justify-content: space-between;
}

.game-group-details {
  display: block;
  width: 15rem;
  text-align: center;
}

.game-group-details-controls {
  margin-top: 4rem;
}

.game-group-poster {
  display: block;
  max-width: 30rem;
  height: 25rem;
  object-fit: scale-down;
}

.pill {
  background: white;
  border-radius: 1.4rem;
  font-size: 1rem;
  padding: 0.4rem 2rem;
  color: black;
  font-weight: 300;
  text-transform: uppercase;
  display: inline-block;
  margin: 0.4rem 0;
  opacity: 0.7;
  transition: 0.5s;
}

.pill-big {
  border-radius: 1.6rem;
  font-size: 1.2rem;
  font-weight: 600;
}

.pill:hover {
  opacity: 1;
}

.blue {
  background-color: #0087ff;
  color: white;
  opacity: 1;
}

.blue:hover {
  background-color: white !important;
  color: #0087ff !important;
}

.title {
  width: 15rem;
  height: 15rem;
  object-fit: scale-down;
}

.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family:
    'Quicksand',
    'Source Sans Pro',
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    'Helvetica Neue',
    Arial,
    sans-serif;
  display: block;
  font-weight: 300;
  font-size: 3rem;
  color: #8e9caa;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}

p {
  margin-top: 1rem;
  font-size: 0.8rem
}

.description {
  margin: 1rem;
  padding: 1rem;
  background-color: rgba(0,0,0,.31);
  color: white;
}

article {
  font-size: 0.8rem;
  padding: 1rem;
}

.description-additional {
  font-size: 0.8rem;
  padding: 1rem;
}

.tag {
  border-radius: 0.5rem;
  border: 0.1rem solid white;
  text-align: center;
  padding: 0 0.4rem;
  display: inline-block;
  margin: 0 0 0.2rem 0.4rem;
  height: 1rem;
  line-height: 0.8rem;
}

.empty {
  display: inline-block;
}

.bold { 
  font-weight: 900;
}

.link {
  color: white;
  border: 0.1rem solid;
  text-decoration: none;
  border-radius: 0.5rem;
  padding: 00.5rem;
  display: inline-flex;
  margin: 1rem 0;
}
</style>
