<template>
  <div class="flex w-full justify-around">
    <template v-if="currentDevice == ''">
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
                {{ extraInfo[tile.row]?.diceText}}
              </div>
            </template>
            <div class="border-2 aspect-square relative" :class="tile.col === 8 ? 'red-line-after': ''" ></div>
            <div v-if="tile.col === 13" class="finish-line bg-green-500"></div>
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
            </div>
          </div>
        </div>
    
        <div class="p-4">
          <button @click="rollDice" class="bg-blue-500 text-white px-4 py-2 rounded">Roll Dice</button>
          <p class="mt-4 text-xl">Result: <i :class="diceIcon(diceNum1)"></i> <i :class="diceIcon(diceNum2)"></i></p>
        </div>
      </div>
    </template>
  
    <template v-if="currentDevice == ''">
      <div class="main-mobile max-w-[414px]">
        <div class="top-row">
          <div class="relative tiles-container grid grid-cols-[repeat(5,minmax(0,1fr))] gap-2 overflow-hidden">
            <template v-for="(order, index) in specialOrders" :key="index">
              <div class="bg-amber-700 border border-amber-200 aspect-[16/9] p-1">
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
                <div class="border aspect-square relative bg-green-700" style="text-wrap: nowrap;">
                  <strong class="text-xl"><small>x</small>{{ bet.odds }}</strong>
                  <div v-if="bet.penalty !== 0" class="absolute w-1/3 h-1/2 bottom-0 right-0 bg-red-500">
                    <small>-{{ bet.penalty }}</small>
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
            <div class="col-span-2"></div>
            <div class="text-center">
              <small>青が１着</small>
              <div class="border aspect-square relative bg-cyan-200 text-black bold text-left p-1" style="text-wrap: nowrap;">
                <strong class="text-3xl"><small>x</small>5</strong>
                <div class="absolute w-1/3 h-1/3 bottom-0 right-0 bg-red-500 text-center">
                  <small>-1</small>
                </div>
              </div>
            </div>
            <div class="text-center">
              <small>黄が１着</small>
              <div class="border aspect-square relative bg-amber-200 text-black bold text-left p-1" style="text-wrap: nowrap;">
                <strong class="text-3xl"><small>x</small>3</strong>
                <div class="absolute w-1/3 h-1/3 bottom-0 right-0 bg-red-500 text-center">
                  <small>-1</small>
                </div>
              </div>
            </div>
            <div class="text-center">
              <small>赤が１着</small>
              <div class="border aspect-square relative bg-pink-200 text-black bold text-left p-1" style="text-wrap: nowrap;">
                <strong class="text-3xl"><small>x</small>2</strong>
                <div class="absolute w-1/3 h-1/3 bottom-0 right-0 bg-red-500 text-center">
                  <small>-1</small>
                </div>
              </div>
            </div>
            <div class="text-center">
              <small style="text-wrap: nowrap">7が5着以下</small>
              <div class="border aspect-square relative bg-amber-950 bold text-left p-1" style="text-wrap: nowrap;">
                <strong class="text-3xl"><small>x</small>4</strong>
              </div>
            </div>
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

      currentDevice: 'iPad',
      specialOrders: [],
    }
  },
  mounted(){
    console.clear();
    this.horseData = this.initHorseData();

    this.currentDevice = '';
    this.specialOrders = this.initSpecialOrders();
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
          horseIndex: row
        });
      }
      return baseData;
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

      if(matchedHorse.position >= 15){
        matchedHorse.position = 15;
        this.winner = matchedHorse
        alert("We have a winner!");
      }

      this.previousOccurance = matchedHorse.occurance;
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
    diceIcon(num) {
      return `fa-solid fa-dice-${['one','two','three','four','five','six'][num - 1]}`
    },
  }
}
</script>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    padding: .5rem;
    min-height: 100vh;

    background-color: #4b5563;
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
</style>
