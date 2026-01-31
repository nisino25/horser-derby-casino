<template>
  <div class="flex w-full justify-around">

    <template v-if="currentDevice == 'tablet' || currentDevice == 'pc'">
      <div class="main-tablet max-w-[1024px]">
        <div class="relative tiles-container grid grid-cols-[repeat(17,minmax(0,1fr))] gap-4 overflow-hidden">
          <template v-for="(tile, index) in baseTiles" :key="index">
            <template v-if="tile.col === 0">
              <div 
                class="flex justify-center items-center border border-4 border-black rounded-md font-bold text-black"
                :class="'bg-' + extraInfo[tile.row]?.color + '-300'"
              >
                {{ extraInfo[tile.row]?.text}}
              </div>
              <div class="flex justify-center items-center border border-2 text-sm">
                
              </div>
            </template>
            <div class="border-2 aspect-square relative" :class="tile.col === 8 ? 'red-line-after': ''" ></div>
            <div v-if="tile.col === 13" class="border-2 finish-line bg-green-500"></div>
          </template>
    
    
          <!-- Pieces layer -->
          <div class="absolute top-0 left-[calc(4.5%+1rem)] w-[calc(88%+1rem)] h-[calc(100%+1rem)] pointer-events-none"> 
            <div
              v-for="(horse, horseIndex) in horseData"
              :key="horse.horseIndex"
              class="piece absolute text-[1.75rem] transition-all duration-300"
              :style="{
                top: `${horseIndex * (100 / 9) - .25}%`,
                left: `${horse.position * 6.67 + .4}%`
              }"
            >
              <i :class="['fa-solid', 'fa-horse', `text-${extraInfo[horseIndex]?.color}-200`]"></i>
              <span class="absolute z-10 top-1/4 left-1/3 -translate-x-1/2  text-sm font-bold text-black">
                {{ extraInfo[horseIndex]?.diceText}}
              </span>
            </div>
          </div>
        </div>
    
        <div class="p-4">

          <template v-if="winner && currentRound == totalRound">
            <strong class="mr-4">
              Game is over. {{ totalWinner.name }} ({{ totalWinner.balance }}) won the entire game!
            </strong>
            <button
                @click="confirmNewRound"
                class="group flex items-center gap-2 bg-gradient-to-r from-green-500 to-emerald-600
                      hover:from-green-600 hover:to-emerald-700
                      text-white font-semibold px-5 py-2.5 rounded-xl
                      shadow-lg hover:shadow-xl active:scale-95
                      transition-all duration-200"
            >
                <i class="fa-solid fa-play text-sm group-hover:translate-x-0.5 transition-transform"></i>
                <span class="whitespace-nowrap">Start New Round</span>
            </button>

          </template>
          <button
              v-else-if="winner"
              @click="startNewRound()"
              class="bg-green-500 hover:bg-green-600 text-white px-4 py-2 rounded mr-3 items-center gap-2"
          >
              <i class="fa-solid fa-play"></i>
              Start New Round
          </button>
          <button
              v-else-if="!autoInterval"
              @click="startAuto"
              class="bg-green-500 hover:bg-green-600 text-white px-4 py-2 rounded mr-3 items-center gap-2"
          >
              <i class="fa-solid fa-play"></i>
              Start
          </button>

          <button
              v-else
              @click="stopAuto"
              class="bg-red-500 hover:bg-red-600 text-white px-4 py-2 rounded mr-3 items-center gap-2"
              :disabled="!autoInterval"
          >
              <i class="fa-solid fa-stop"></i>
              Stop
          </button>

          <button v-if="!winner" @click="rollDice" class="bg-blue-500 text-white px-4 py-2 rounded">Roll Dice</button>

          <div>
            <strong
                v-if="winner"
                class="block mt-4 px-6 py-4 rounded-xl
                      bg-yellow-100 text-yellow-800 border border-yellow-300
                      text-lg font-bold shadow-lg text-center"
            >
                🏆 Winner: {{ extraInfo[winner.horseIndex]?.diceText }} 🏆
            </strong>

            <strong
              v-else-if="hasCrossedRedLine"
              class="block mt-4 px-4 py-2 rounded-lg
                    bg-red-50 text-red-700 border border-red-300
                    text-sm font-semibold shadow-sm"
            >
              赤線を越えた馬が3頭出たため、ベット終了です。
            </strong>
          </div>
          <p class="mt-4 text-3xl">
            <strong>{{ currentRound }} / {{ totalRound }} Round</strong> - Result: <i :class="diceIcon(diceNum1)"></i> <i :class="diceIcon(diceNum2)"></i>
          </p>
        </div>
      </div>
    </template>
  
    <template v-if="currentDevice == 'mobile' || currentDevice == 'pc'">
      <div class="main-mobile max-w-[414px]">
        <div class="top-row">
          <div class="relative tiles-container grid grid-cols-[repeat(5,minmax(0,1fr))] gap-2 overflow-hidden">
            <template v-for="(order, index) in specialOrders" :key="index">
              <div class="bg-amber-700 border border-amber-200 aspect-[16/9] p-1 relative" @click="placeSpecialBet(order)">
                <div class="center">
                  <div class="flex justify-between w-full text-sm">
                    <div class="w-full text-center">
                      <i :class="['fa-solid', 'fa-horse', `text-${extraInfo[order.behind]?.color}-200`]"></i>
                      <div>
                        <small>{{ extraInfo[order.behind].diceText }}</small>
                      </div>
                    </div>
                    <div><strong>||</strong></div>
                    <div class="w-full text-center">
                      <i :class="['fa-solid', 'fa-horse', `text-${extraInfo[order.ahead]?.color}-200`]"></i>
                      <div>
                        <small>{{ extraInfo[order.ahead].diceText }}</small>
                      </div>
                    </div>
                  </div>
                  <hr>
                  <div class="flex justify-between w-full text-sm px-1">
                    <div>x<strong>{{ order.odds }}</strong></div>
                    <div class="text-black">-<strong>{{ order.penalty }}</strong></div>
                  </div>
                </div>

                <div
                  v-if="order?.placedBet"
                  class="absolute bottom-1 left-7 w-6 aspect-square rounded-full text-xs text-black flex items-center justify-center"
                  :class="order.color"
                >
                  {{ order?.placedBet }}
                </div>
                <!-- {{ order }} -->
                
              </div>
            </template>
          </div>
        </div>
        <div class="middle-row">
          <div class="relative grid grid-cols-[1fr_1fr_22.5px_1fr_1fr_22.5px_1fr_1fr_1fr] gap-2 items-center overflow-hidden mt-4">
            <div class="border col-span-2 text-center">1–3着</div>
            <div></div>
            <div class="border col-span-2 text-center">1-2着</div>
            <div></div>
            <div class="border col-span-3 text-center">1着</div>
            <template v-for="(info, index) in extraInfo" :key="index">
              <template v-for="(bet, betIndex) in info.betList" :key="betIndex">
                <div 
                  class="border aspect-square relative bg-green-700 px-1" 
                  style="text-wrap: nowrap;"
                  @click="placeBet(myPlayerIndex, bet)"
                  >
                  <strong class="text-lg"><small>x</small>{{ bet.odds }}</strong>
                  <div v-if="bet.penalty !== 0" class="absolute w-1/3 h-1/2 bottom-0 right-0 bg-red-500">
                    <small>-{{ bet.penalty }}</small>
                  </div>
                  <div
                    v-if="bet.placedBet"
                    class="absolute bottom-0 left-2 w-5 h-5 rounded-full text-xs text-black flex items-center justify-center"
                    :class="bet.color"
                  >
                    {{ bet.placedBet }}
                  </div>

                </div>
                <div 
                  v-if="betIndex == 1 || betIndex == 3" 
                  class="w-full text-center flex flex-col justify-around items-center gap-1"
                  >
                  <i :class="['fa-solid', 'fa-horse', `text-${info.color}-200`]"></i>
                  <div class="leading-none"><small>{{ info.diceText }}</small></div>
                </div>
              </template>
            </template>
          </div>
        </div>
        <div class="bottom-row">
          <div class="relative grid grid-cols-[1fr_1fr_1fr_1fr_1fr_1fr] gap-2 items-center overflow-hidden mt-4">
            <div class="col-span-2">
              <div class="grid grid-cols-[1fr_1fr_1fr] gap-1 w-[80%]">
                <template v-for="(bet, betIndex) in myPlayer.bets" :key="betIndex">
                    <div 
                        class="w-7 aspect-square flex items-center justify-center rounded-full text-black font-bold shadow-md m-1"
                        :class="[
                            myPlayer.coinColor,
                            selectedBetIndex === betIndex ? 'scale-110 brightness-110 animate-pulse' : ''
                        ]"
                        @click="selectedBetIndex = betIndex"
                    >
                        {{ bet }}
                    </div>
                </template>
              </div>

            </div>
            <template v-for="(bet, index) in topBets" :key="index">
              <div class="text-center whitespace-nowrap relative" @click="placeSpecialBet(bet)">
                <small>{{ bet.name }}</small>
                <div :class="bet.bgColor" class="border aspect-square relative text-black bold text-left p-1 ">
                  <strong class="text-3xl" :class="index == 3 ? 'text-white' : ''"><small>x</small>{{ bet.odds }}</strong>
                  <div class="absolute w-1/3 h-1/3 bottom-0 right-0 bg-red-500 text-center" v-if="bet.penalty > 0">
                    <small>-{{ bet.penalty }}</small>
                  </div>
                </div>
                  <div
                      v-if="bet?.placedBet"
                      class="absolute bottom-1 left-2 w-6 aspect-square rounded-full text-xs text-black flex items-center justify-center"
                      :class="bet.color"
                    >
                      {{ bet?.placedBet }}
                  </div>
              </div>
            </template>
          </div>
        </div>
        <div class="players-row">
          <div class="relative tiles-container grid grid-cols-[repeat(5,minmax(0,1fr))] gap-4  mt-4">
            <template v-for="(player, playerIndex) in players" :key="playerIndex">
              <div class="playerInfo" :id="'player-'+player.name">
                <div class="player-box">
                    <div class="name-container text-black py-1" :class="player.coinColor">
                        <p>{{ player.name }} : {{ player.balance }}</p>
                    </div>
                    <div class="player-image-container relative">
                        <div class="temp-image">
                            <div v-html="regenerate(player.randomString)"></div>
                        </div>
                    </div>
                </div>
              </div>
            </template>
          </div>
        </div>
      </div>
    </template>

  </div>

</template>

<script>
  export default {
    name: 'App',
    components: {
    },
    data() {
      return {
        baseTiles: this.initBaseTiles(),
        horseData: [],
        extraInfo: [
          {
            boost: 3, text: "+3", diceText: "2,3", color: "cyan",
            betList: [
              {odds: 4, penalty: 4},{odds: 4, penalty: 3},
              {odds: 5, penalty: 4},{odds: 5, penalty: 3},
              {odds: 7, penalty: 2},{odds: 8, penalty: 2},{odds: 9, penalty: 2},
            ]
          },
          {
            boost: 3, text: "+3", diceText: "4", color: "cyan",
            betList: [
              {odds: 3, penalty: 1},{odds: 3, penalty: 0},
              {odds: 4, penalty: 1},{odds: 4, penalty: 0},
              {odds: 5, penalty: 1},{odds: 6, penalty: 0},{odds: 7, penalty: 0},
            ]
          },
          {
            boost: 2, text: "+2", diceText: "5", color: "amber",
            betList: [
              {odds: 2, penalty: 3},{odds: 2, penalty: 0},
              {odds: 2, penalty: 2},{odds: 3, penalty: 2},
              {odds: 4, penalty: 2},{odds: 4, penalty: 0},{odds: 5, penalty: 0},
            ]
          },
          {
            boost: 1, text: "+1", diceText: "6", color: "pink",
            betList: [
              {odds: 1, penalty: 2},{odds: 1, penalty: 0},
              {odds: 2, penalty: 5},{odds: 2, penalty: 4},
              {odds: 3, penalty: 2},{odds: 3, penalty: 1},{odds: 3, penalty: 0},
            ]
          },
          {
            boost: 0, text: "0", diceText: "7", color: "black",
            betList: [
              {odds: 1, penalty: 3},{odds: 1, penalty: 1},
              {odds: 2, penalty: 6},{odds: 2, penalty: 5},
              {odds: 3, penalty: 4},{odds: 3, penalty: 3},{odds: 3, penalty: 2},
            ]
          },
          {
            boost: 1, text: "+1", diceText: "8", color: "pink",
            betList: [
              {odds: 1, penalty: 2},{odds: 1, penalty: 0},
              {odds: 2, penalty: 5},{odds: 2, penalty: 4},
              {odds: 3, penalty: 2},{odds: 3, penalty: 1},{odds: 3, penalty: 0},
            ]
          },
          {
            boost: 2, text: "+2", diceText: "9", color: "amber",
            betList: [
              {odds: 2, penalty: 3},{odds: 2, penalty: 0},
              {odds: 2, penalty: 2},{odds: 3, penalty: 2},
              {odds: 4, penalty: 2},{odds: 4, penalty: 0},{odds: 5, penalty: 0},
            ]
          },
          {
            boost: 3, text: "+3", diceText: "10", color: "cyan",
            betList: [
              {odds: 3, penalty: 1},{odds: 3, penalty: 0},
              {odds: 4, penalty: 1},{odds: 4, penalty: 0},
              {odds: 5, penalty: 1},{odds: 6, penalty: 0},{odds: 7, penalty: 0},
            ]
          },
          {
            boost: 3, text: "+3", diceText: "11,12", color: "cyan",
            betList: [
              {odds: 4, penalty: 4},{odds: 4, penalty: 3},
              {odds: 5, penalty: 4},{odds: 5, penalty: 3},
              {odds: 7, penalty: 2},{odds: 8, penalty: 2},{odds: 9, penalty: 2},
            ]
          },
        ],
        diceNum1: null,
        diceNum2: null,
        diceSum: null,
        previousOccurance: null,
        hasCrossedRedLine: false,
        winner: null,

        currentDevice: '',
        specialOrders: [],
        topBets: [
          {name: "青が１着", colorName: "青", bgColor: "bg-cyan-200", odds:5, penalty: 1, condition: "best"},
          {name: "黄が１着", colorName: "黄", bgColor: "bg-amber-200", odds:3, penalty: 1,  condition: "best"},
          {name: "赤が１着", colorName: "赤", bgColor: "bg-pink-200", odds:2, penalty: 1,  condition: "best"},
          {name: "7が5着以下", bgColor: "bg-amber-950", odds:4, penalty: 0,  condition: "worst"}
        ],

        players:[],
        autoInterval: null,

        currentRound: 1,
        totalRound: 4,

        myPlayerName: "Nozo",
        selectedBetIndex: null,

      }
    },
    mounted(){
      console.clear();

      const width = window.innerWidth

      if (width >= 1024) {
          this.currentDevice = 'pc'
      } else if (width >= 768) {
          this.currentDevice = 'tablet'
      } else {
          this.currentDevice = 'mobile'
      }

      // this.horseData = this.initHorseData();

      // this.specialOrders = this.initSpecialOrders();

      // this.players = this.initPlayers();

      // this.randomBetAll();

    },
    methods: {
      initBaseTiles(){
        const tiles = [];
        for(let row=0; row<9; row++){
          for(let col=0; col<14; col++){
            tiles.push({row,col});
          }
        }
        return tiles;
      },
      initHorseData(){
        const baseData = [];
        for(let row=0; row<9; row++){
          const occurance =this.extraInfo[row]?.diceText
          .split(",")
          .map(num => parseInt(num));

          baseData.push({
            boost: this.extraInfo[row].boost,
            occurance,
            position: 0,
            horseIndex: row,
            rank: null,
          });
        }
        return baseData;
      },
      initSpecialOrders(){
        // 0 - [2,3]
        // 1 - 4
        // 2 - 5
        // 3 - 6
        // 4 - 7
        // 5 - 8
        // 6 - 9
        // 7 - 10
        // 8 - [11,12]

        const baseData = [
          {behind: 2, ahead: 0, odds: 2, penalty: 1},
          {behind: 2, ahead: 8, odds: 2, penalty: 1},
          {behind: 2, ahead: 1, odds: 3, penalty: 3},
          // {behind: [2,6], ahead: 5, odds: 3, penalty: 3},
          {behind: 4, ahead: 6, odds: 4, penalty: 4},

          {behind: 5, ahead: 7, odds: 3, penalty: 2},
          {behind: 5, ahead: 0, odds: 4, penalty: 3},
          {behind: 5, ahead: 8, odds: 4, penalty: 3},
          // {behind: [0,1,7,8], ahead: 5, odds: 2, penalty: 3},

          {behind: 6, ahead: 0, odds: 3, penalty: 3},
          {behind: 6, ahead: 7, odds: 2, penalty: 1},
        ]

        // make a copy so original is untouched
        const pool = [...baseData];

        // Fisher–Yates shuffle
        for (let i = pool.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [pool[i], pool[j]] = [pool[j], pool[i]];
        }

        // take first 5 (all unique)
        return pool.slice(0, 5);
      },
      initPlayers() {
          const names = ["Becca", "Nozo", "Lebron", "Curry"]
          const coinColors = [
              'bg-yellow-400',
              'bg-red-500',
              'bg-blue-400',
              'bg-purple-400',
              'bg-amber-500'
          ]

          return names.map((name, index) => ({
              name,
              balance: 0,
              score: 0,            // make sure to initialize score too
              bets: [2, 3, 3, 4, 5],
              randomString: this.generateAvatarSeed(),
              coinColor: coinColors[index % coinColors.length]
          }))
      },

      generateAvatar() {
        const randomString = Math.random().toString();
        return {
            avatar: window.multiavatar(randomString),
            randomString: randomString
        };
      },
      generateAvatarSeed() {
        return Math.random().toString(36).slice(2);
      },

      regenerate(randomString) {
          return window.multiavatar(randomString);
      },
      rollDice(){
        this.diceNum1 = Math.floor(Math.random() * 6) + 1;
        this.diceNum2 = Math.floor(Math.random() * 6) + 1;
        this.diceSum = this.diceNum1+this.diceNum2;

        const matchedHorse = this.horseData.find(horse =>
          horse.occurance.includes(this.diceSum)
        );

        matchedHorse.position += 1;

        if(matchedHorse.occurance == this.previousOccurance){
          matchedHorse.position += matchedHorse.boost;
        }

        let redLineCount = 0
        for(let horse in this.horseData){
          if(this.horseData[horse].position > 9){
            redLineCount += 1;
          }
        }

        if(redLineCount == 3 && !this.hasCrossedRedLine){
          this.hasCrossedRedLine = true;
        }

        if(matchedHorse.position >= 15){
          matchedHorse.position = 15;
          this.winner = matchedHorse
          this.calculateScores()
        }

        this.previousOccurance = matchedHorse.occurance;
      },

      diceIcon(num) {
        return `fa-solid fa-dice-${['one','two','three','four','five','six'][num - 1]}`
      },

      placeBet(playerIndex, targetBet) {
          if (this.hasCrossedRedLine) return;
          if (targetBet.placedBet) return;

          const player = this.players[playerIndex];

          if (player.bets.length === 0) return;
          

          let betValue;

          // Use selectedBetIndex if set (can be 0!)
          if (this.selectedBetIndex !== null && this.selectedBetIndex !== undefined) {
              betValue = player.bets.splice(this.selectedBetIndex, 1)[0];
              this.selectedBetIndex = null;
          } else {
              // Default to the last bet
              betValue = player.bets.pop();
          }

          // Apply the bet to the target tile
          targetBet.placedBet = betValue;
          targetBet.color = player.coinColor;
          targetBet.playerIndex = playerIndex;
      },
      placeSpecialBet(targetBet) {

        if (this.hasCrossedRedLine) return;
        if (targetBet.placedBet) return;
        // if (!this.selectedBetIndex ) return
        if (this.myPlayer.bets.length === 0) return;

          
        let betValue;

        betValue = this.myPlayer.bets.splice(this.selectedBetIndex, 1)[0];
        this.selectedBetIndex = null;

        // Apply the bet to the target tile
        targetBet.placedBet = betValue;
        targetBet.color = this.myPlayer.coinColor;
        targetBet.playerIndex = this.myPlayerIndex;
      },

      randomBetAll() {
          const middleIndex = Math.floor(this.extraInfo.length / 2)

          this.players.forEach((player, playerIndex) => {
              if (player.name === this.myPlayerName) return
              while (player.bets.length > 0) {
                  const availableBets = []

                  // collect free tiles with weight
                  this.extraInfo.forEach((info, infoIndex) => {
                      info.betList.forEach(bet => {
                          if (!bet.placedBet) {
                              // weight based on distance to middle index
                              const distance = Math.abs(infoIndex - middleIndex)
                              const weight = Math.max(this.extraInfo.length - distance, 1) // higher weight for middle
                              for (let i = 0; i < weight; i++) {
                                  availableBets.push(bet)
                              }
                          }
                      })
                  })

                  if (availableBets.length === 0) return

                  const randomBet = availableBets[Math.floor(Math.random() * availableBets.length)]
                  this.placeBet(playerIndex, randomBet)
              }
          })
      },
      startAuto() {
          // prevent double start
          if (this.autoInterval) return
          this.rollDice()

          // const frequency = 250
          const frequency = 25
          

          this.autoInterval = setInterval(() => {
              if (this.winner) {
                  clearInterval(this.autoInterval)
                  this.autoInterval = null
                  return
              }

              this.rollDice()
          }, frequency)
      },
      stopAuto() {
          if (this.autoInterval) {
              clearInterval(this.autoInterval)
              this.autoInterval = null
          }
      },

      getHorseRanks() {
          // clone + sort
          const sorted = [...this.horseData]
              .sort((a, b) => b.position - a.position)

          // assign rank (1-based)
          sorted.forEach((horse, index) => {
              horse.rank = index + 1
          })
      },
      isWinningBet(betIndex, horseRank) {
          // Top 3
          if (betIndex <= 1) {
              return horseRank <= 3
          }

          // Top 2
          if (betIndex <= 3) {
              return horseRank <= 2
          }

          // Winner only
          return horseRank === 1
      },
      calculateScores() {
          this.getHorseRanks()

          this.extraInfo.forEach(info => {
              const horse = this.horseData.find(
                  h => h.occurance.join(",") === info.diceText
              )

              info.betList.forEach((bet, betIndex) => {
                  if (!bet.placedBet) return

                  const player = this.players[bet.playerIndex]
                  const amount = bet.placedBet

                  const win = this.isWinningBet(betIndex, horse?.rank)

                  if (!player.balance) player.score = 0

                  if (win) {
                      player.balance += amount * bet.odds
                  } else {
                      player.balance -= bet.penalty
                  }
              })

              


              
          })

          this.specialOrders.forEach(order => {
              const behindHorse = this.horseData[order.behind]
              const aheadHorse = this.horseData[order.ahead]

              
              if (!behindHorse || !aheadHorse) return

              const isSuccess = aheadHorse.rank < behindHorse.rank

              if (!order.placedBet) return

              const player = this.players[order.playerIndex]
              const amount = order.placedBet

              // if (!player.balance) player.balance = 0

              if (isSuccess) {
                player.balance += amount * order.odds
              } else {
                player.balance -= order.penalty
              } 
          })

          this.topBets.forEach(bet => {
            if (!bet.placedBet) return

            const player = this.players[bet.playerIndex]
            const amount = bet.placedBet

            let isSuccess = false

            if (bet.condition === 'best') {
                const colorMap = {
                    '青': 'cyan',
                    '黄': 'amber',
                    '赤': 'pink'
                }

                const targetColor = colorMap[bet.colorName]
                if (!targetColor) return

                // 1️⃣ find indexes (should be exactly 2)
                const indexes = []
                this.extraInfo.forEach((info, i) => {
                    if (info.color === targetColor) {
                        indexes.push(i)
                    }
                })

                // if (indexes.length !== 2) return
                console.log(indexes)

                // 2️⃣ get horses using those indexes
                const horseA = this.horseData[indexes[0]]
                const horseB = this.horseData[indexes[1]]
                const horseC = this.horseData[indexes[2]]
                const horseD = this.horseData[indexes[3]]

                // if (!horseA || !horseB) return

                // 3️⃣ check rank
                isSuccess = horseA.rank === 1 || horseB.rank === 1 || horseC?.rank === 1 || horseD?.rank === 1
            }


            if (bet.condition === 'worst') {
                // number-based: "7が5着以下"
                // const horseIndex = Number(bet.name[0])
                const horse = this.horseData[4]
                if (!horse) return

                isSuccess = horse.rank >= 5
            }

            console.log(bet.name)
            if (isSuccess) {
                player.balance += amount * bet.odds
            } else {
                player.balance -= bet.penalty
            }
            console.log(player.balance)

          })

          // this.players.forEach(player => {
          //   if(player.balance < 0) player.balance = 0
          // })
      },

      startNewRound(){
        this.currentRound++;
        this.winner = null;
        this.hasCrossedRedLine = false;

        this.horseData = this.initHorseData();

        this.extraInfo.forEach((info) => {
            info.betList.forEach(bet => {
                bet.placedBet = null;
                bet.color = null;
                bet.playerIndex = null;
            })
        })

        this.specialOrders.forEach((specialBet) => {
          specialBet.placedBet = null;
          specialBet.color = null;
          specialBet.playerIndex = null;
        })

        this.topBets.forEach((bet) => {
          bet.placedBet = null;
          bet.color = null;
          bet.playerIndex = null;
        })

        this.players.forEach(player => {
          player.bets =  [2, 3, 3, 4, 5];
        })


        this.randomBetAll();

      },

      confirmNewRound() {
        const ok = window.confirm('Start a new round?\nCurrent progress will be lost.')
        if (!ok) return

        location.reload()
      },
      



      
    },
    computed: {
        totalWinner() {
            if (!this.players || this.players.length === 0) return null

            // find the player with the highest balance
            return this.players.reduce((maxPlayer, player) => {
                return (player.balance > (maxPlayer?.balance || 0)) ? player : maxPlayer
            }, null)
        },

        myPlayer() {
            return this.players.find(player => player.name === this.myPlayerName)
        },

        myPlayerIndex(){
          return this.players.findIndex(player => player.name === this.myPlayerName);
        },

    }

  }
</script>

<style>
  * {
    touch-action: manipulation;
    -webkit-user-select: none;

    padding: 0;
    margin: 0;
  }
  #app {
    overflow: hidden;
    height: 100%;
    margin: 0;
    padding: 0;

    font-family: Avenir, Helvetica, Arial, sans-serif;
    padding: .5rem;
    /* min-height: 100vh; */

    background-color: #4b5563;
    background: #4b5563;

    color: #f9fafb; /* Tailwind gray-50 */
    /* -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    margin-top: 60px; */
  }

  .red-line-after::after{
    content: "";
    position: absolute;
    top: 50%;
    left: calc(100% + .65em);
    transform: translateX(-50%) translateY(-50%);
    width: .25em;
    height: 200%;
    background-color: red;
  }

  .playerInfo{
    position: relative;
    text-align: center;
    font-size: .7em;
  }

  .playerInfo .player-box{
    position: relative;
    border: 1px solid black;
    transition: border-color 0.5s ease-in-out, box-shadow 0.5s ease-in-out;
    
    width: 80px;
  }

  .playerInfo .name-container {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 1.5em; /* Adjust based on your font-size and line-height */
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap; /* Prevent line breaks */
    text-align: center; /* Center text horizontally */
    padding: 5px 3px;
  }
  .playerInfo .name-container p {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap; /* Prevent line breaks */
    width: 100%; /* Ensure the span takes the full width */
    text-align: center; /* Center text horizontally */
  }
  .playerInfo .player-image-container{
    display: block;
    width: 100%;
    aspect-ratio: 1;
    /* background: red; */
  }
  .playerInfo .player-image-container .temp-image{
    display: block;
    margin: auto;
    /* width: calc(100% - 2px); */
    width: 100%;
    padding: 5px;
    aspect-ratio: 1;

    box-sizing: border-box;

    background: #5D6D7E;

    border: 2px solid transparent;
  }

  .playerInfo span{
    font-size: 2em;
    line-height: 1.25;

    font-weight: bold;

    text-shadow: 2px 2px 5px rgba(0, 0, 0, 0.5);
  }

</style>
