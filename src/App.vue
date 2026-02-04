<template>
  <div class="flex w-full justify-around">


    <div v-if="currentPage === 'before'">
        <div class="bg-white p-5 rounded-lg shadow-md w-[85%] max-w-[400px] m-auto absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 text-black">

          <div v-if="!roomOption">
            <button @click="createARoom()" class="bg-[#3581B8] text-white  block w-1/2 mx-auto p-[10px] rounded">Create a room</button>
            <hr class="my-3">
            <button @click="roomOption = 'join'; retriveCode();" class="bg-[#13563B] text-white block w-1/2 mx-auto p-[10px] rounded">Join a room</button>
          </div>
          <template v-if="roomOption && !roomCode">
            <h2>Type your name</h2>
              <div class="flex items-center mb-3 border border-gray-300 rounded overflow-hidden">
              <input
                type="text"
                class="flex-grow p-2 outline-none"
                v-model="username"
                placeholder="Enter your username"
              >
              <button
                class="p-2 text-gray-600 hover:text-gray-900"
                @click="username = getRandomName()"
              >
                <i class="fa-solid fa-shuffle"></i>
              </button>
            </div>

              <div class="w-[95%] mx-auto grid grid-cols-5 gap-3 justify-between mb-5">
                <div v-for="(avatar, index) in avatars" :key="index" @click="randomString = avatar.randomString" v-html="avatar.avatar"  :style="{ opacity:  randomString !== avatar.randomString ? '0.6' : '1' }"></div>

                <div class="text-3xl text-gray-600 flex justify-center items-center text-center" @click="generateAvatars('female')">
                  <i class="fas fa-sync"></i>
                </div>
              </div>
              <div class="flex items-center mb-2" v-if="roomOption === 'join'">
                <input type="number" v-model="tempRoomcode" placeholder="Type room code" class="flex-grow p-2 border border-gray-300 rounded">
              </div>
              <!-- <button v-if="readyToPlay" @click="randomName()" class="add-button">ランダム</button> -->
              <button @click="roomOption = null" class="bg-[#B83A4B] text-white block mx-auto p-[10px] rounded w-1/2 mb-2">Back</button>
              <button @click="username = getRandomName();" class="bg-[black] text-white block mx-auto p-[10px] rounded mb-2 w-1/2">Random Name</button>
              <!-- <button v-if="readyToPlay && roomOption === 'create'" @click="createARoom()" class="bg-[#3581B8] text-white  block  mx-auto p-[10px] rounded w-1/2 mb-2">Create</button> -->
              <button v-if="tempRoomcode >= 10000 && tempRoomcode <= 99999 && readyToPlay && roomOption === 'join'" @click="joinARoom()" class="bg-[#3581B8] text-white  block mx-auto p-[10px] rounded w-1/2">Join</button>
          </template>

          <template v-if="roomOption && roomCode">
            <template  v-if="isHost">
              <h2>You are host</h2>
              Game Speed
              <input type="number" v-model="frequency" class="mb-4">
              <hr>
            </template>
              
            <h2 v-if="!isHost">Welcome {{ username }}!</h2>
            <p>Room code: <strong class="font-size: 2.5em; color: crimson; margin-right: 5px; font-weight: bold;">{{ roomCode }}</strong></p>
            <hr>
            <template v-for="(player, index) in players" :key="index">
              <div class="player-list flex items-center gap-2 my-2">
                <span>{{index +1}}.</span>
                <div v-html="regenerate(player.randomString)"></div>
                <p>{{ player.name }}</p>
              </div>
            </template>
            <button v-if="players?.length >= 2 && isHost"  @click="closeTheRoom()" class="bg-[#3581B8] text-white  block  mx-auto p-[10px] rounded w-1/2 mb-2">Close room</button>
          </template>
        </div>
      </div>

    <div v-if="currentPage === 'game'">
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
                <span class="absolute z-10 left-1/3 -translate-x-1/2  text-sm font-bold text-black" style="top: 25%; text-shadow: 0 0 4px rgba(255, 255, 255, 0.8),0 0 8px rgba(255, 255, 255, 0.6);">
                  {{ extraInfo[horseIndex]?.diceText}}
                </span>
              </div>
            </div>
          </div>
      
          <div class="p-4">
    
            <template v-if="winner && currentRound == totalRound">
              <strong class="mr-4">
                Game is over. {{ totalWinner?.name }} ({{ totalWinner?.balance }}点) won the entire game!
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
                  <span class="whitespace-nowrap">Start New Game</span>
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
                <div 
                  class="bg-amber-700 border border-amber-200 aspect-[16/9] p-1 relative"
                  @click="placeSpecialBet(order)"
                  :class="{
                    'cannotBet': !winner && (hasCrossedRedLine || order.placedBet),
                    'blink-win': winner && order.isSuccess,
                  }"
                  >
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
                </div>
              </template>
            </div>
          </div>
          <div class="middle-row">
            <div class="relative grid grid-cols-[1fr_1fr_22.5px_1fr_1fr_22.5px_1fr_1fr_1fr] gap-1 items-center overflow-hidden mt-2">
              <div class="border col-span-2 text-center">1–3着</div>
              <div></div>
              <div class="border col-span-2 text-center">1-2着</div>
              <div></div>
              <div class="border col-span-3 text-center">1着</div>
              <template v-for="(info, index) in extraInfo" :key="index">
                <template v-for="(bet, betIndex) in info.betList" :key="betIndex">
                  <div 
                    class="border aspect-square relative px-1 bg-green-700" 
                    style="text-wrap: nowrap;"
                    :class="{
                      'cannotBet': !winner && (hasCrossedRedLine || bet.placedBet),
                      'blink-win': winner && bet.isSuccess,
                    }"

                    @click="placeBet(myPlayerIndex, bet)"
                    >
                    <div class="w-[80%]">
                      <strong class="text-lg block leading-none"><small>x</small>{{ bet.odds }}</strong>
                      <small class="bg-red-500 leading-none px-1">-{{ bet.penalty }}</small>
                    </div>
                    <div
                      v-if="bet.placedBet"
                      class="absolute bottom-3 right-0 w-5 aspect-square rounded-full text-xs text-black flex items-center justify-center"
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
            <div class="relative grid grid-cols-[1fr_1fr_1fr_1fr_1fr_1fr] gap-2 items-center overflow-hidden mt-2">
              <div class="col-span-2">
                <div class="grid grid-cols-[1fr_1fr_1fr] gap-1 w-[80%]">
                  <template v-for="(bet, betIndex) in myPlayer?.bets" :key="betIndex">
                      <div 
                          class="w-7 aspect-square flex items-center justify-center rounded-full text-black font-bold shadow-md m-1"
                          :class="[
                              myPlayer.coinColor,
                              selectedBetIndex === betIndex ? 'brightness-100' : 'brightness-50'
                          ]"
                          @click="this.selectedBetIndex = this.selectedBetIndex === betIndex ? null : betIndex"
                      >
                          {{ bet }}
                      </div>
                  </template>
                </div>
    
              </div>
              <template v-for="(bet, index) in topBets" :key="index">
                <div 
                  class="text-center whitespace-nowrap relative" 
                  @click="placeSpecialBet(bet)"
                  >
                  <small>{{ bet.name }}</small>
                  <div 
                    :class="[
                        bet.bgColor,
                        { cannotBet: !winner && (hasCrossedRedLine || bet.placedBet) },
                        { 'blink-win': winner && bet.isSuccess},
                    ]"

                    class="border aspect-square relative text-black bold text-left p-1 "
                    >
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
            <div class="relative tiles-container grid grid-cols-[repeat(4,minmax(0,1fr))] gap-3  mt-2">
              <template v-for="(player, playerIndex) in players" :key="playerIndex">
                <div class="playerInfo" :id="'player-'+player.name" >
                  <div class="player-box" :class="player.name == username ? 'border-2 border-yellow-400 shadow-[0_0_10px_rgba(255,215,0,0.7)]' : 'border border-gray-300'">
                      <div class="name-container text-black py-1" :class="player.coinColor">
                          <p>{{ player.name }}</p>
                      </div>
                      <div class="player-image-container relative">
                          <div class="temp-image">
                              <div v-html="regenerate(player.randomString)"></div>
                          </div>
                          <template v-if="winner">
                            <div class="absolute top-0 left-0 w-full h-full flex flex-col justify-center items-center bg-black bg-opacity-50 text-white text-center p-2 text-2xl font-bold">
                              <strong :class="diffClass(player)">
                                {{ player.balanceArr[player.balanceArr.length - 1] - player.balanceArr[player.balanceArr.length - 2] }}
                              </strong>
                            </div>
                          </template>
                      </div>
                  </div>
                  <div class="w-full border text-black" :class="player.coinColor">
                    <div class="flex justify-between px-2" style="text-wrap: nowrap;">
                      <span style="font-size: 0.75rem;">{{ player.balance }}P</span>
                      <span style="font-size: 0.75rem;">残{{ player.bets.length }}</span>
                    </div>
                  </div>
                </div>
              </template>
            </div>
          </div>
        </div>
      </template>
    </div>

    
  </div>
  
</template>

<script>

  import db from './firebase.js';
  import { randomNames } from './name.js';

  export default {
    name: 'App',
    components: {
    },
    data() {
      return {
        baseTiles: [],
        horseData: [],
        extraInfo: [],
        specialOrders: [],
        topBets: [],

        diceNum1: null,
        diceNum2: null,
        diceSum: null,
        previousOccurance: null,
        hasCrossedRedLine: false,
        winner: null,

        currentDevice: '',

        players:[],
        autoInterval: null,

        currentRound: 1,
        totalRound: 4,

        myPlayerName: "Nozo",
        selectedBetIndex: null,

        currentPage: 'before',

        // ----------------

        defaultNumber: 2,
        maxPlayerNumber: 6,
        randomNames,

        firebaseRoomName: 'horse-rooms',
        roomOption: null,
        roomCode: null,
        tempRoomcode: null,
        generalData: null,

        username: null,
        onlineStatus: '',

        gameResults: [],
        previousGameResults: [],

        pickedRandomString: null,
        avatars: [],
        randomString: 0,

        isCheckingNow: false,
        gameMessage: "",
        isHost: false,
        developingMode: false,

        frequency: 2500,
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

      this.generateAvatars();
      this.currentPlayerIndex = 0

      this.username = this.getRandomName();


      // this.developingMode = true;
      if(this.developingMode){
        this.frequency = 25
      }




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

        this.updateHorse();

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

          this.updateBetsAndScore();
      },
      placeSpecialBet(targetBet) {

        if (this.hasCrossedRedLine) return;
        if (targetBet.placedBet) return;
        // if (!this.selectedBetIndex ) return
        if (this.myPlayer.bets.length === 0) return;

        let betValue;

        if (!this.selectedBetIndex){
          betValue = this.myPlayer.bets.pop();
        }else{
          betValue = this.myPlayer.bets.splice(this.selectedBetIndex, 1)[0];
          this.selectedBetIndex = null;
        }


        // Apply the bet to the target tile
        targetBet.placedBet = betValue;
        targetBet.color = this.myPlayer.coinColor;
        targetBet.playerIndex = this.myPlayerIndex;

        this.updateBetsAndScore();
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
          // this.frequency = 3000
          

          this.autoInterval = setInterval(() => {
              if (this.winner) {
                  clearInterval(this.autoInterval)
                  this.autoInterval = null
                  return
              }

              this.rollDice()
          }, this.frequency)
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

                  const player = this.players[bet.playerIndex] || {balance: 0}
                  const amount = bet.placedBet

                  const win = this.isWinningBet(betIndex, horse?.rank)

                  if (!player.balance) player.score = 0

                  if (win) {
                    bet.isSuccess = true;
                      player.balance += amount * bet.odds;
                  } else {
                    bet.isSuccess = false;
                      player.balance -= bet.penalty;
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
                order.isSuccess = true
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

            if (isSuccess) {
              bet.isSuccess = true;
                player.balance += amount * bet.odds
            } else {
                player.balance -= bet.penalty
            }
            console.log(player.balance)

          })

          this.players.forEach(player => {
            player.balanceArr.push(player.balance);
          })

          this.updateBetsAndScore();
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
              bet.isSuccess = false;
            })
        })

        this.specialOrders.forEach((specialBet) => {
          specialBet.placedBet = null;
          specialBet.color = null;
          specialBet.playerIndex = null;
          specialBet.isSuccess = false;
        })

        this.topBets.forEach((bet) => {
          bet.placedBet = null;
          bet.color = null;
          bet.playerIndex = null;
          bet.isSuccess = false;
        })

        this.players.forEach(player => {
          player.bets =  [2, 3, 3, 4, 5];
        })

        this.updateBetsAndScore();


        // this.randomBetAll();

      },

      confirmNewRound() {
        const ok = window.confirm('Start a new round?\nCurrent progress will be lost.')
        if (!ok) return

        location.reload()
      },

      // ----------------------
      randomName(){
          this.username = this.getRandomName();
      },
      retriveCode(){
        this.tempRoomcode = localStorage.getItem('latestRoomCode') || 'No room code found'
        if(this.devSkip) this.joinARoom()
      },
      async createARoom() {
        if (this.roomCode) return;


        if(!this.developingMode){
          const ok = window.confirm('Start a new room?')
          if (!ok) return
        }
        // if(!this.username) return;

        let isUnique = false;

        // Generate a unique room code
        while (!isUnique) {
          this.tempRoomcode = Math.floor(10000 + Math.random() * 90000);
          const docRef = db.collection(this.firebaseRoomName).doc(`${this.tempRoomcode}`);
          const doc = await docRef.get();
          if (!doc.exists) {
            isUnique = true;
          }
        }

        // console.log(this.roomCode)
        this.roomCode = this.tempRoomcode
        localStorage.setItem('latestRoomCode', this.roomCode)

        this.baseTiles = this.initBaseTiles(),

        this.extraInfo = [
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

        this.horseData = this.initHorseData();

        this.specialOrders = this.initSpecialOrders();

        this.topBets = [
          {name: "青が１着", colorName: "青", bgColor: "bg-cyan-200", odds:5, penalty: 1, condition: "best"},
          {name: "黄が１着", colorName: "黄", bgColor: "bg-amber-200", odds:3, penalty: 1,  condition: "best"},
          {name: "赤が１着", colorName: "赤", bgColor: "bg-pink-200", odds:2, penalty: 1,  condition: "best"},
          {name: "7が5着以下", bgColor: "bg-amber-950", odds:4, penalty: 0,  condition: "worst"}
        ];

        const ref = db.collection(this.firebaseRoomName)
        ref.doc(`${this.roomCode}`).set({
          games: JSON.stringify([{ gameStatus: 'waiting' }]),
          players: [],
          onlineStatus: 'waiting',
          horseData : this.horseData,
          extraInfo : this.extraInfo,
          specialOrders : this.specialOrders,
          topBets : this.topBets,
          hasCrossedRedLine: false,
        })

        this.roomOption = 'create'
        this.onlineStatus = 'waiting'
        this.isHost = true
        await this.reciveTheData()
      },

      async joinARoom() {

        const docRef = db.collection(this.firebaseRoomName).doc(`${this.tempRoomcode}`);

        try {
          const doc = await docRef.get();
          if (doc.exists) {
            if(doc.data().onlineStatus == 'playing') return alert('This room is closed.')

            this.players = doc.data().players;
            this.onlineStatus = doc.data().players;

            if (!this.players.includes(this.username)) {
              this.players.push(
                {
                  name:this.username,
                  isHost:false,
                  randomString: this.randomString,

                  balance: 0,
                  balanceArr: [0],
                  score: 0,            // make sure to initialize score too
                  bets: [2, 3, 3, 4, 5],
                }
              );
              this.roomCode = this.tempRoomcode

            }

            await docRef.update({
              players: this.players,
            });
            this.reciveTheData();
          } else {
            console.log('No such document!');
          }
        } catch (error) {
          console.log('Error getting document:', error);
        }
      },

      reciveTheData(){
        
        db.collection(this.firebaseRoomName).doc(`${this.roomCode}`)
        .onSnapshot((doc) => {

          this.generalData = doc.data()
          
          // joining room and wait until it closes
          // if(this.currentPage == 'before'){
          this.onlineStatus = this.generalData?.onlineStatus
          this.players = this.generalData?.players

          this.extraInfo = this.generalData?.extraInfo
          this.specialOrders = this.generalData?.specialOrders
          this.topBets = this.generalData?.topBets
          
          // this.horseData = this.generalData?.horseData
          if(this.isHost == false){
            this.hasCrossedRedLine = this.generalData?.hasCrossedRedLine
            this.winner = this.generalData?.winner
          }



          if(this.onlineStatus == 'playing' || this.onlineStatus == 'distributing') {
            this.deck = this.generalData.deck;
            this.publicPile = this.generalData.publicPile;


            this.lastSubmitBy = this.generalData?.lastSubmitBy


            // check if the game is overr


            this.currentPlayerIndex = this.generalData.currentPlayerIndex
            this.currentPage = 'game'
            localStorage.setItem('latestRoomCode', null);

            
            this.isRevolutionGoing = this.generalData.isRevolutionGoing
            this.isTempRevolutionGoing = this.generalData.isTempRevolutionGoing

            this.gameResults = this.generalData.gameResults

            

          }
        
        })
      },

      async closeTheRoom(){

        if(this.players.length < 2) return

        const coinColors = [
            'bg-yellow-400',
            'bg-red-500',
            'bg-blue-400',
            'bg-purple-400',
            'bg-amber-500'
        ]

        this.players.forEach((player,index) =>{
          player.coinColor = coinColors[index]
        })

        this.currentPage = 'game';

        // this.gameResults = []

        this.onlineStatus = 'playing'
        const ref = db.collection(this.firebaseRoomName)
        ref.doc(`${this.roomCode}`).update({
          // gameResults: this.gameResults,
          // previousGameResults: this.previousGameResults,
          // deck: this.deck,
          hasCrossedRedLine: this.hasCrossedRedLine,
          winner: '',
          totalWinner: '',
          players: this.players,
          onlineStatus: this.onlineStatus,
          // currentPlayerIndex: this.currentPlayerIndex,
          // publicPile: this.publicPile,

        })
      },
      updateBetsAndScore(){
        const ref = db.collection(this.firebaseRoomName)
        ref.doc(`${this.roomCode}`).update({
          players: this.players,
          extraInfo: this.extraInfo,
          specialOrders: this.specialOrders,
          topBets: this.topBets,
          hasCrossedRedLine: this.hasCrossedRedLine,
          winner: false,
        })
      },
      updateHorse(){
        const ref = db.collection(this.firebaseRoomName)
        ref.doc(`${this.roomCode}`).update({
          horseData: this.horseData,
          hasCrossedRedLine: this.hasCrossedRedLine,
          winner: this.winner,
        })
      },


      generateAvatars() {
        this.tempAvatarCode = null;

        this.avatars = [1, 2, 3, 4, 5,6,7,8,9].map(() => {
          const randomString = Math.random().toString();
          return {
            avatar: window.multiavatar(randomString),
            randomString: randomString
          };
        });
      },
      getRandomName() {
        let randomName;
        do {
          const randomIndex = Math.floor(Math.random() * this.randomNames.length);
          randomName = this.randomNames[randomIndex];
        } while (this.players.some(player => player.name === randomName));

        // this.generateAvatars();

        return randomName;
      },

      devSkip(mode){
        if(this.developingMode == false) return
        if(mode == 'create'){
          this.createARoom()
        }else if(mode == 'join'){   
          this.joinARoom();
        }
      },

      diffClass(player) {
        const arr = player.balanceArr
        const diff = arr[arr.length - 1] - arr[arr.length - 2]

        if (diff > 0) return 'text-green-400'
        if (diff < 0) return 'text-red-400'
        return 'text-gray-300'
      }
      
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
            return this.players.find(player => player.name === this.username)
        },

        myPlayerIndex(){
          return this.players.findIndex(player => player.name === this.username);
        },

        readyToPlay() {
          const namePattern = /^[^\s!@#$%^&*(),.?":{}|<>]+$/;

          // Check if the username is valid
          return namePattern.test(this.username) && this.username?.trim() !== '';
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
  html,body {
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
    /* border: 1px solid black; */
    transition: border-color 0.5s ease-in-out, box-shadow 0.5s ease-in-out;
    
    /* width: 80px; */
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

    /* text-shadow: 2px 2px 5px rgba(0, 0, 0, 0.5); */
  }

  .cannotBet::before{
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0,0,0,0.5);
    z-index: 10;
  }
  @keyframes blink-win {
      0%, 100% {
          background-color: #facc15; /* yellow-400 */
      }
      50% {
          background-color: grey;
      }
  }

  .blink-win {
    animation: blink-win 1.5s infinite;
  }


</style>
