<template>
  <div>
    <div class="container mw-100">
      <div class="row">
        <template v-if="state === 'game'">
          <div class="col-12 top-panel d-flex align-items-center justify-content-between">
            <div class="counter rounded-circle d-flex align-items-center justify-content-center h2 m-0">{{ counter }}</div>
            <button class="btn btn-warning btn-lg" @click="restartGame">Restart</button>
          </div>
          <div class="col left">
            <div class="row justify-content-center h-100 align-items-center">
              <div class="col-lg-8 col-xl-6">
                <div class="grid">
                  <div class="grid-item" v-for="card in shuffledCards" :key="card.id" @click="toggleCard(card)" :class="{open: card.id === activeCards.first?.id || card.id === activeCards.second?.id || card.found}">
                    <div class="item-backface"></div>
                    <img :src="`img/${card.src}.jpg`" alt="" class="w-100 item-img">
                  </div>
                </div>
              </div>
            </div>
          </div>
        </template>
        <div class="col right" v-if="state === 'finish'">
          <div class="row justify-content-center">
            <div class="col-lg-8 col-xl-7">
              <h1 class="text-center mb-4">STATS</h1>
              <table class="table bg-light">
                <tr>
                  <th>#</th>
                  <th>Name</th>
                  <th>Score</th>
                </tr>
                <tr v-for="(result, index) in sortResults()" :key="index">
                  <td>{{ index + 1 }}</td>
                  <td>{{ result.name }}</td>
                  <td>{{ result.score }}</td>
                </tr>
              </table>
              <div class="text-center">
                <button class="btn btn-warning btn-lg" @click="restartGame">Restart</button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Wellcome Message Modal -->
    <div id="wellcomeMessage" class="modal fade" data-backdrop="static" data-keyboard="false">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h3 class="modal-title">Worldskills Americas</h3>
            <button type="button" class="close" data-dismiss="modal">×</button>
          </div>
          <div class="modal-body">
            <p>Welcome to the Guatemala Memory Game</p>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-primary" @click="startGame">Start Game</button>
          </div>
        </div>
      </div>
    </div>

    <div id="finishMessage" class="modal fade" data-backdrop="static" data-keyboard="false">
      <div class="modal-dialog">
        <form class="modal-content" @submit.prevent="saveResult">
          <div class="modal-header">
            <h3 class="modal-title">Great job</h3>
          </div>
          <div class="modal-body">
            <label for="user_name">Name</label>
            <input type="text" v-model="userName" id="user_name" class="form-control" placeholder="Name">
          </div>
          <div class="modal-footer">
            <button class="btn btn-primary" :disabled="!userName">OK</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      state: 'init',
      shuffledCards: [],
      foundCards: [],
      activeCards: {first: null, second: null},
      results: [],
      counter: 0,
      userName: null
    }
  },
  computed: {
    notFoundCards: function () {
      return this.shuffledCards.filter(v => !v.found)
    }
  },
  methods: {
    startGame: function () {
      this.state = 'game'
      this.counter = 0
      this.initializeCards()
      window.$("#wellcomeMessage").modal('hide');
    },
    initializeCards: function () {
      let cards = []
      for (let i = 1; i <= 3; i++) {
        cards.push(`memo-${i < 10 ? '0' : ''}${i}`)
      }
      this.shuffledCards = [...cards].concat([...cards]).sort(() => Math.random() - 0.5).map((item, index) => ({id: index + 1, src: item, open: false}))
    },
    toggleCard: function (card) {
      if (!card.open) {
        if (!this.activeCards.first) this.activeCards.first = card
        else if (!this.activeCards.second) {
          this.counter++
          this.activeCards.second = card
          if (this.activeCards.first.src === this.activeCards.second.src) {
            this.activeCards.first.found = true
            this.activeCards.second.found = true
            this.activeCards = {first: null, second: null}
            if (!this.notFoundCards.length) this.finishGame()
          } else {
            setTimeout(() => {
              this.activeCards = {first: null, second: null}
            }, 1000)
          }
        }
      }
    },
    finishGame: function () {
      window.$("#finishMessage").modal('show');
    },
    saveResult: function () {
      this.state = 'finish'
      window.$("#finishMessage").modal('hide');
      this.results.push({name: this.userName, score: this.counter})
      localStorage.setItem('_g_results', JSON.stringify(this.results))
    },
    restartGame: function () {
      this.state = 'init'
      window.$("#wellcomeMessage").modal('show');
    },
    sortResults: function () {
      let resultsGroups = {}
      this.results.sort((a, b) => a.score < b.score ? 1 : (a.score > b.score ? -1 : 0)).forEach(result => {
        (resultsGroups[result.score] = resultsGroups[result.score] || []).push(result)
      })
      let results = []
      for (let result in resultsGroups) {
        results.push({
          score: result,
          name: resultsGroups[result].map(v => v.name).filter((v, i, a) => a.indexOf(v) === i).join(', ')
        })
      }
      return results.slice(0, 10)
    }
  },
  mounted() {
    window.$("#wellcomeMessage").modal('show');
    this.results = localStorage.getItem('_g_results') ? JSON.parse(localStorage.getItem('_g_results')) : []
  }
}
</script>

<style>
body{
  background-color: #EEEDE9;
}

.container .left{
  min-height: calc(100vh - 100px);
  background-color: #f59081;
}
.top-panel {
  height: 100px;
}
.counter {
  width: 70px;
  height: 70px;
  border: 3px solid #2bb1ff;
}

.container .right{
  min-height: 100vh;
  padding-top: 50px;

  background-color: #2bb1ff;
}
.grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  grid-gap: 20px;
}
.grid-item {
  position: relative;
  cursor: pointer;
}
.item-backface {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: #8def8d;
  transition: .4s;
  backface-visibility: hidden;
}
.item-img {
  backface-visibility: hidden;
  position: relative;
  transition: .4s;
}
.grid-item:not(.open) .item-img {
  transform: rotateY(180deg);
}
.grid-item.open .item-backface {
  transform: rotateY(180deg);
}
</style>
