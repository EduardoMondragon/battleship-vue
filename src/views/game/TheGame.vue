<template>
  <div class="column is-12 main">
    <div v-if="!winner" class="box" :style="{'background-image': `url(${imageUrl(skin)})`}">
      <div class="columns">
        <div class="column is-10 has-text-left">
          <div v-if="!isPlaying()" class="has-text-warning">
            <p>Aguardando {{player === 1 ? nameThem : nameUs}} jogar ... ... ...</p>
          </div>
          <div v-if="isPlaying()" class="has-text-info">
            <p>{{player === 1 ? nameUs : nameThem}}, faça sua jogada !</p>
          </div>
        </div>
        <div class="column is-2" v-if="isPlaying()">
          <p :class="timer <= 5 ? 'has-text-danger' : 'has-text-success' ">{{timer}} segundos restantes ..</p>
        </div>
      </div>
      <div class="columns">
        <div class="column is-4 them-table">
          <div v-if="player === 1" class="">
            <table-board ref="tableBoardUs" @onSelect="select" :isPlaying="isPlaying()" :itsMe="player === 1" :columns="columnsUs"></table-board>
          </div>
          <div v-if="player === 2">
            <table-board ref="tableBoardThem" @onSelect="select" :isPlaying="isPlaying()" :itsMe="player === 2" :columns="columnsThem"></table-board>
          </div>
        </div>
        <div class="column is-3">
          <div class="text-area">
            <div v-for="msg in messages" class="has-text-left">
              {{msg.user}}: {{ msg.text }}
            </div>
          </div>
          <div class="field has-addons">
            <div class="control">
              <input type="text" class="input" placeholder="Digite ..." v-model="msgUser">
            </div>
            <div class="control">
              <a class="button is-success" @click="addNewMsg('fsdf')">
                Enviar
              </a>
            </div>
          </div>
        </div>
        <div class="column is-5">
          <div v-if="player === 1">
            <table-board ref="tableBoardThem" @onSelect="select" :isPlaying="isPlaying()" :itsMe="player === 2" :columns="columnsThem"></table-board>
          </div>
          <div v-if="player === 2">
            <table-board ref="tableBoardUs" @onSelect="select" :isPlaying="isPlaying()" :itsMe="player === 1" :columns="columnsUs"></table-board>
          </div>
        </div>
      </div>
    </div>
    <div v-if="winner" class="box hero is-widescreen finish-box">
      <div v-if="winner === player" class="columns">
        <div class="box hero column is-4 is-offset-4">
          <p class="has-text-info">
            Winner !!! :)
          </p>
          <p class="has-text-info">
            Sua pontuação foi: {{scorePoints()}}
          </p>
          <a class="button is-dark" @click="saveScore()">Novo Jogo</a>
        </div>
      </div>
      <div v-if="winner !== player" class="columns">
        <div class="box hero column is-4 is-offset-4">
          <p class="has-text-danger">
            Looser !!! :(
          </p>
          <a class="button is-dark" @click="newGame()">Novo Jogo</a>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import TableBoard from '@/components/TableBoard'
  import FleetBoard from '@/components/FleetBoard'

  import * as ds from 'deepstream.io-client-js'
  import axios from 'axios'

  export default {
    data () {
      return {
        ds: ds('wss://013.deepstreamhub.com?apiKey=9606832e-3795-4c89-ac0c-05e02b996285').login(),
        player: null,
        record: null,
        recordPlayer: null,
        recordFleet: null,
        recordMessage: null,
        timer: 20,
        messages: [],
        msgUser: '',
        fleetUs: {},
        fleetThem: {},
        nameUs: '',
        nameThem: '',
        idShip: 0,
        timerPoints: 0,
        selectedsPoints: 0,
        shipsFleet: [
          {id: 1, src: 'ship1.png', length: 1},
          {id: 2, src: 'ship2.png', length: 2},
          {id: 3, src: 'ship3.png', length: 3},
          {id: 4, src: 'ship4.png', length: 4},
          {id: 5, src: 'ship5.png', length: 5}
        ],
        orientation: 'H',
        columnsThem: [
          {y: 0, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 1, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 2, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 3, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 4, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 5, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 6, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 7, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 8, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 9, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]}
        ],
        columnsUs: [
          {y: 0, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 1, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 2, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 3, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 4, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 5, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 6, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 7, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 8, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]},
          {y: 9, rows: [{x: 0, hit: false}, {x: 1, hit: false}, {x: 2, hit: false}, {x: 3, hit: false}, {x: 4, hit: false}, {x: 5, hit: false}, {x: 6, hit: false}, {x: 7, hit: false}, {x: 8, hit: false}, {x: 9, hit: false}]}
        ],
        playerPlaying: null,
        winner: false,
        skin: '',
        recordSkin: null
      }
    },
    components: {
      TableBoard,
      FleetBoard
    },
    methods: {
      saveScore () {
        const params = {
          user: this.player === 1 ? this.nameUs : this.nameThem,
          ranking: this.scorePoints()
        }
        axios.post('http://localhost:3001/api/rank', params)
          .then(
            this.$router.push('awaitPlayers')
          )
          .catch(err => {
            alert(err.response.data)
          })
      },
      newGame () {
        this.$router.push({name: 'awaitPlayers'})
      },
      scorePoints () {
        const timerPoints = this.timerPoints * 5
        const selectedsPoints = this.selectedsPoints * 50
        return 10000 - timerPoints - selectedsPoints
      },
      imageUrl (img) {
        return require('@/assets/templates/themes/' + img)
      },
      setTimerPoints () {
        if (!this.winner) {
          this.timerPoints++
          setTimeout(this.setTimerPoints, 1000)
        }
      },
      setTimer () {
        if (!this.winner) {
          if (this.isPlaying()) {
            this.timer--
            setTimeout(this.setTimer, 1000)
          } else {
            this.timer = 20
            setTimeout(this.setTimer, 1000)
          }

          if (this.timer === 0) {
            if (this.playerPlaying === 1 && this.player === 1) {
              this.recordPlayer.set('playerPlaying', 2)
            } else {
              this.recordPlayer.set('playerPlaying', 1)
            }
            alert('Demorou demais :I ')
          }
        }
      },
      addNewMsg () {
        this.messages.push({user: this.player === 1 ? this.nameUs : this.nameThem, text: this.msgUser})
        this.recordMessage.set('messages', this.messages)
        this.msgUser = ''
      },
      select (args) {
        this.selectedsPoints++
        if (this.playerPlaying === 1 && this.player === 1) {
          this.recordPlayer.set('playerPlaying', 2)
        } else {
          this.recordPlayer.set('playerPlaying', 1)
        }
        this.handleThem()
        this.handleUs()

        let fleet
        if (this.player === 1) {
          fleet = this.fleetThem
        } else {
          fleet = this.fleetUs
        }

        for (const prop in fleet) {
          for (let x = 0; x < fleet[prop].length; x++) {
            const ship = fleet[prop][x]
            if (ship.x === args.x && ship.y === args.y) {
              fleet[prop][x].status = false
              alert('Acertou miseravi')
              let rest = 0
              for (let x = 0; x < fleet[prop].length; x++) {
                if (fleet[prop][x].status) {
                  rest++
                }
              }
              if (rest === 0) {
                let theEnd = true
                for (const propI in fleet) {
                  for (let i = 0; i < fleet[propI].length; i++) {
                    if (fleet[propI][i].status) {
                      theEnd = false
                    }
                  }
                }
                if (theEnd) {
                  this.recordPlayer.set('winner', this.player)
                  alert('Biiiuurrr ! Você ganhou !!')
                } else {
                  alert('Já era esse navio')
                }
              }
            }
          }
        }
      },
      isPlaying () {
        return this.player === this.playerPlaying
      },
      handleUs () {
        this.record.set('columnsUs', this.columnsUs)
      },
      handleThem () {
        this.record.set('columnsThem', this.columnsThem)
      }
    },
    created () {
      this.recordSkin = this.ds.record.getRecord(`battleship/skin/${this.$route.params.game}`)
      this.recordFleet = this.ds.record.getRecord(`battleship/fleet/${this.$route.params.game}`)
      this.record = this.ds.record.getRecord(`battleship/game/${this.$route.params.game}`)
      this.recordPlayer = this.ds.record.getRecord(`battleship/player/${this.$route.params.game}`)
      this.recordMessage = this.ds.record.getRecord(`battleship/msg/${this.$route.params.game}`)
      this.player = this.$route.params.player

      if (this.player === 1) {
        this.record.set('columnsUs', this.$route.params.columns)
        this.columnsUs = this.$route.params.columns

        this.recordFleet.set('fleetUs', this.$route.params.fleet)
        this.fleetUs = this.$route.params.fleet
        this.recordFleet.set('nameUs', this.$route.params.playerName)
        this.nameUs = this.$route.params.playerName
      } else {
        this.record.set('columnsThem', this.$route.params.columns)
        this.columnsThem = this.$route.params.columns

        this.recordFleet.set('fleetThem', this.$route.params.fleet)
        this.fleetThem = this.$route.params.fleet
        this.recordFleet.set('nameThem', this.$route.params.playerName)
        this.nameThem = this.$route.params.playerName
      }

      this.recordPlayer.set('playerPlaying', 1)
      this.playerPlaying = 1

      this.record.subscribe(values => {
        this.columnsThem = values.columnsThem
        this.columnsUs = values.columnsUs
      })

      this.recordPlayer.subscribe(values => {
        this.playerPlaying = values.playerPlaying
        this.winner = values.winner
      })

      this.recordFleet.subscribe(values => {
        this.fleetThem = values.fleetThem
        this.fleetUs = values.fleetUs

        this.nameThem = values.nameThem
        this.nameUs = values.nameUs
      })

      this.recordMessage.set('messages', [])
      this.recordMessage.subscribe(values => {
        this.messages = values.messages
      })

      this.setTimer()

      this.skin = this.$route.params.skinSelected
      if (this.skin) {
        this.recordSkin.set('skin', this.skin)
      }
      this.recordSkin.subscribe(values => {
        this.skin = values.skin
      })

      this.setTimerPoints()
    }
  }
</script>

<style lang="sass" scoped>
  .icon-edit
    font-size: 24px
    color: #ffa726
    cursor: pointer
  .fa-thumbs-o-down
    font-size: 62px
    color: #e34120
  .them-table
    font-size: 12px
  .text-area
    height: 81%
    margin-bottom: 20px
    max-height: 81%
    overflow: scroll
  .main
    margin-top: -20px
  .finish-box
    background-image: url("./../../assets/templates/finish.jpg")
    height: 600px
</style>
