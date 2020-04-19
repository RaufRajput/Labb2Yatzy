<template>
  <div id="game">
    <div class="container">
      <br>
      <div class="header">
        <h1>Welcome to Yatzy by Rauf, the Yatzy for the solo's</h1>
      </div>
      <div class="flex">
        <div class="inner-flex">
          <div class="boxx one">
            <p>{{ rollText() }}</p>
            <div>
              <span v-for="dice in all_dice" :key="dice.dice_id">
                <span class="dice-size">
                  <i class="fas pointed-cursor" :class="[{ 'has-text-info': dice.selected }, diceClass(dice.value)]" @click="toggleSelected(dice)"></i>
                </span>
              </span>
            </div>
            <p>
              <button class="btn" @click="rollAll()" :disabled="rollLimitReached() || gameComplete()">
                <i class="fas fa-dice" :class="[{ 'fa-spin': !rollLimitReached() && !gameComplete() }]"></i>
                Roll the dices
              </button>
            </p>
          </div>
        </div>
        <div class="boxx">
          <div >
            <table class="table">
              <thead>
                <tr>
                  <td class="section-title" width="20%">Upper Section</td>
                  <td class="section-title" width="60%">How To instructions</td>
                  <td class="section-title" width="20%">Score</td>
                </tr>
              </thead>
              <tbody>
                <tr class="pointed-cursor" v-for="line in scorecard.upper_section" :key="line.label" @click="scoreRoll(line)">
                  <td>{{ line.label }} <i class="fas fa-check" v-if="line.scored"></i></td>
                  <td>{{ line.howto }}</td>
                  <td>{{ line.score }}</td>
                </tr>
              </tbody>
            </table>
            <table >
              <thead>
                <tr>
                  <td class="section-title" width="20%">Lower Section</td>
                  <td class="section-title" width="60%">How To instructions</td>
                  <td class="section-title" width="20%">Score</td>
                </tr>
              </thead>
              <tbody>
                <tr class="pointed-cursor" v-for="line in scorecard.lower_section" :key="line.label" @click="scoreRoll(line)">
                  <td>{{ line.label }} <i class="fas fa-check " v-if="line.scored"></i></td>
                  <td>{{ line.howto }}</td>
                  <td>{{ line.score }}</td>
                </tr>
              </tbody>
            </table>
            <table>
              <thead>
                <tr>
                  <td class="section-title" width="20%">Totals</td>
                  <td class="section-title" width="60%"></td>
                  <td class="section-title" width="20%">Score</td>
                </tr>
              </thead>
              <tbody>
                <tr>
                  <td colspan="2" class="text-left">Upper Section Total</td>
                  <td>{{ scorecard.upper_total }}</td>
                </tr>
                <tr>
                  <td colspan="2" class="text-left">Lower Section Total</td>
                  <td>{{ scorecard.lower_total }}</td>
                </tr>
                <tr>
                  <td colspan="2" class="text-left">Grand Total</td>
                  <td>{{ scorecard.grand_total }}</td>
                </tr>
              </tbody>
            </table>
            <button class="btn mt-15" @click="newGame()">New Game</button>
            <br><br><br>
            <div class="footer">
            <p>Developed by Rauf fall semester 2020 ITHS. School project in webb applications </p>
          </div>
            <br>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import { mapActions, mapState } from 'vuex'
  export default {
    name: 'game',
    computed: {
      ...mapState(
        'scorecard', [
          'scorecard'
        ]
      )
    },
    data () {
      return {
        roll_count: 0,
        all_dice: [
          {
            dice_id: 1,
            value: null,
            selected: false
          },
          {
            dice_id: 2,
            value: null,
            selected: false
          },
          {
            dice_id: 3,
            value: null,
            selected: false
          },
          {
            dice_id: 4,
            value: null,
            selected: false
          },
          {
            dice_id: 5,
            value: null,
            selected: false
          }
        ]
      }
    },
    mounted () {
      this.fetchScorecard()
      this.fetchHighscores()
    },
    methods: {
      ...mapActions(
        'highscores', [
          'saveHighscore', 'fetchHighscores'
        ]
      ),
      ...mapActions(
        'scorecard', [
          'updateScorecard', 'fetchScorecard', 'resetScorecard'
        ]
      ),
      roll (dice) {
        dice.value = Math.floor(Math.random() * 6) + 1
      },
      rollAll () {
        for (var dice of this.all_dice.filter(d => d.selected === false)) {
          this.roll(dice)
        }

        this.roll_count++
      },
      toggleSelected (dice) {
        if (this.roll_count === 0) {
          return
        }

        dice.selected = !dice.selected
      },
      rollLimitReached () {
        return this.roll_count >= 3
      },
      diceClass (value) {
        switch (value) {
          case 1: return 'fa-dice-one'
          case 2: return 'fa-dice-two'
          case 3: return 'fa-dice-three'
          case 4: return 'fa-dice-four'
          case 5: return 'fa-dice-five'
          case 6: return 'fa-dice-six'
          default: return 'fa-square'
        }
      },
      rollText () {
        if (this.gameComplete()) {
          return 'Game Complete! Thanks for Playing!'
        }

        switch (this.roll_count) {
          case 1: return 'First Roll'
          case 2: return 'Second Roll'
          case 3: return 'Final Roll. Please Score.'
          default: return 'Click Roll to Get Started!'
        }
      },
      resetDice () {
        this.roll_count = 0

        for (var dice of this.all_dice) {
          dice.value = null
          dice.selected = false
        }
      },
      addGrandTotal () {
        this.scorecard.grand_total = this.scorecard.upper_total + this.scorecard.lower_total
      },
      scoreRoll (line) {
        if (this.roll_count === 0) {
          return
        }

        if ((line.scored || line.id === 'bonus') && line.id !== 'yahtzee') {
          return
        }

        switch (line.id) {
          case 'aces':
            this.scoreIndividualUpper(line, 1)
            break
          case 'twos':
            this.scoreIndividualUpper(line, 2)
            break
          case 'threes':
            this.scoreIndividualUpper(line, 3)
            break
          case 'fours':
            this.scoreIndividualUpper(line, 4)
            break
          case 'fives':
            this.scoreIndividualUpper(line, 5)
            break
          case 'sixes':
            this.scoreIndividualUpper(line, 6)
            break
          case 'three_of_a_kind':
            this.scoreXOfAKind(line, 3)
            break
          case 'four_of_a_kind':
            this.scoreXOfAKind(line, 4)
            break
          case 'full_house':
            this.scoreFullHouse(line)
            break
          case 'small_straight':
            this.scoreSmallStraight(line)
            break
          case 'large_straight':
            this.scoreLargeStraight(line)
            break
          case 'yahtzee':
            this.scoreyahtzee(line)
            break
          case 'chance':
            this.scoreChance(line)
            break
          default: return
        }

        // bonus
        var bonus_line = this.scorecard.upper_section.filter(l => l.id === 'bonus')[0]
        if (!bonus_line.scored) {
          if (this.scorecard.upper_total >= 63) {
            bonus_line.score = 35
            this.scorecard.upper_total += bonus_line.score
            bonus_line.scored = true
          }
        }

        line.scored = true
        this.addGrandTotal()
        this.resetDice()
        this.updateScorecard()

        // spara totalen till highscores.
        if (this.gameComplete()) {
          this.saveHighscore(this.scorecard.grand_total)
        }
      },
      scoreIndividualUpper (line, dice_num) {
        for (var dice of this.all_dice) {
          if (dice.value === dice_num) {
            line.score += dice_num
          }
        }

        this.scorecard.upper_total += line.score
      },
      scoreXOfAKind (line, limit) {
        var dice_check = this.buildDiceCheckMap()

        for (var value of dice_check.dice_map_values) {
          if (value >= limit) {
            line.score = dice_check.dice_total
            this.scorecard.lower_total += dice_check.dice_total
          }
        }
      },
      scoreFullHouse (line) {
        var dice_check = this.buildDiceCheckMap()

        var has_2 = false
        var has_3 = false
        for (var value of dice_check.dice_map_values) {
          if (value === 2) { has_2 = true }
          if (value === 3) { has_3 = true }
        }

        if (has_2 && has_3) {
          line.score = 25
          this.scorecard.lower_total += 25
        }
      },
      scoreSmallStraight (line) {
        var dice_check = this.buildDiceCheckArray()

        if (dice_check.length >= 4) {
          dice_check = dice_check.slice(0, 4) // endast 4 tärningar för small straight
          if (
            this.$_.isEqual(dice_check, [1, 2, 3, 4]) ||
            this.$_.isEqual(dice_check, [2, 3, 4, 5]) ||
            this.$_.isEqual(dice_check, [3, 4, 5, 6])) {
              line.score = 30
              this.scorecard.lower_total += 30
          }
        }
      },
      scoreLargeStraight (line) {
        var dice_check = this.buildDiceCheckArray()

        if (dice_check.length === 5) {
          if (
            this.$_.isEqual(dice_check, [1, 2, 3, 4, 5]) ||
            this.$_.isEqual(dice_check, [2, 3, 4, 5, 6])) {
              line.score = 40
              this.scorecard.lower_total += 40
          }
        }
      },
      scoreyahtzee (line) {
        var dice_check = this.buildDiceCheckArray()

        if (dice_check.length === 1) {
          // yatzi två gånger bonus på 100p
          if (line.scored && line.score !== 0) {
            line.score += 100
            this.scorecard.lower_total += 100
          }

          if (!line.scored) {
            line.score = 50
            this.scorecard.lower_total += 50
          }
        }
      },
      scoreChance (line) {
        for (var dice of this.all_dice) {
          line.score += dice.value
        }

        this.scorecard.lower_total += line.score
      },
      buildDiceCheckArray () {
        var dice_check = []
        for (var dice of this.all_dice) {
          if (!dice_check.includes(dice.value)) {
            dice_check.push(dice.value)
          }
        }

        return dice_check.sort()
      },
      buildDiceCheckMap () {
        var dice_map = new Map()
        dice_map.set(1, 0)
        dice_map.set(2, 0)
        dice_map.set(3, 0)
        dice_map.set(4, 0)
        dice_map.set(5, 0)
        dice_map.set(6, 0)

        var dice_total = 0

        for (var dice of this.all_dice) {
          dice_total += dice.value
          dice_map.set(dice.value, dice_map.get(dice.value) + 1)
        }

        return { dice_total: dice_total, dice_map_values: dice_map.values() }
      },
      gameComplete () {
        var upper_complete = false
        if (this.scorecard.upper_section.filter(l => l.id !== 'bonus').filter(l => l.scored === false).length === 0) {
          upper_complete = true
        }

        var lower_complete = false
        if (this.scorecard.lower_section.filter(l => l.scored === false).length === 0) {
          lower_complete = true
        }

        if (upper_complete && lower_complete) {
          return true
        }

        return false
      },
      newGame () {
        this.resetDice()
        this.resetScorecard()
      }
    }
  }
</script>

<style scoped lang="scss">
  #game {
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
  }
  .flex{
    display: flex;
    flex-direction: column;
  }
  .inner-flex{
    display: flex;
    flex-direction: column;
  }
  .boxx{
    background-color: #fffefd;
    box-sizing: border-box;
    padding: 25px;

  }
  .dice-size {
    font-size: 4em;
    padding-left: .10em;
    padding-right: .10em;
  }

  .pointed-cursor {
    cursor: pointer;
  }

  .section-title {
    font-family: 'Pacifico', cursive;
    font-size: 1.25em;
  }
  .btn{
    padding: 8px 18px;
    font-size: 18px;
    color: white;
    border: none;
    background-color: cornflowerblue;
  }
  .pointed-cursor{
    color: #f53b3b;
  }
  .has-text-info{
    color: #484848;;
  }
  table{
    width: 50%;
    margin: auto;
    border: 1px solid #cccccc;
    padding: 15px;
  }
  .fa-check{
    color: cornflowerblue;;
  }
  .mt-15{
    margin-top: 15px;
  }
  .text-left{
    text-align: left;
  }
  td{
    color: #484848;
  }
  .footer {
    position: fixed;
    left: 0;
    bottom: 0;
    width: 100%;
    background-color: #f53b3b;
    color: white;
    text-align: center;
  }
  .header {
    padding: 1px;
    text-align: center;
    background: #f53b3b;
    color: white;
    font-size: 30px;
  }
</style>
