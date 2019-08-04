<template>
  <div class="recommendation-container">
    <div class="recommender-header">
      Personalized Recommender
    </div>

    <div v-if="!isStarted">
      <button class="btn-start" @click="startRecommendation">Start</button>
    </div>


    <div v-else>
      <template v-if="!isCalculated">
        <div class="carousel-container">
          <button class="prev" :disabled="checkLevel(-1)" @click="addLevel(-1)"><img src='../../src/assets/left_arrow.svg'/> </button>
          <button class="next" :disabled="checkLevel(1)" @click="addLevel(1)"><img src="../../src/assets/right_arrow.svg"/></button>
          <div class="recommender-container" v-if="currentRecommender.details">
            <div class="recommender-name">
              {{currentRecommender.details.name}}
            </div>

            <div v-if="currentRecommender.details.type && 'multiselect'.includes(currentRecommender.details.type.toLowerCase())">
              <div v-for="(value, valueIndex) in currentRecommender.details.values">
                <input type="checkbox"
                       v-model="selectedRecommenders"
                       :name="currentRecommender.id"
                       :value="currentRecommender.id+','+value"
                       :id="currentRecommender.id"
                >{{value}}
              </div>
            </div>

            <div v-else-if="currentRecommender.details.type && 'progressbar'.includes(currentRecommender.details.type.toLowerCase())">
              <input type="range" min="0" :max="currentRecommender.details.max">
            </div>

          </div>
        </div>

        <div v-if="isSubmitActive">
          <button @click="submitProcess" class="btn-start">Submit</button>
        </div>

        <div style="margin-top: 20px">
          <button @click="restartProcess" class="btn-start">Restart</button>
        </div>
      </template>
      <template v-else>
        <div>
          {{recommenderResult}}
        </div>
      </template>
    </div>


  </div>
</template>

<script>
  import json from '../assets/recommenders.json';
  import resultJson from '../assets/recommenderResult.json';
  export default {
    name: "landing-page",
    data() {
      return {
        recommenders: json,
        isStarted: false,
        currentLevel:0,
        selectedRecommenders:[],
        isCalculated: false,
        recommenderResult: []
      }
    },
    methods: {
      startRecommendation() {
        this.isStarted = true;
      },
      addLevel(addition) {
        let updatedLevel = this.currentLevel+addition;
        this.currentLevel=updatedLevel;
      },
      restartProcess() {
        this.isStarted = false;
      },
      checkLevel(addition) {
        let updatedLevel = this.currentLevel+addition;
        return !(updatedLevel>=0 && updatedLevel<this.recommenders.levels.length)
      },
      submitProcess() {
        console.log("Selected Recommenders : ",this.selectedRecommenders);
        let recommendedResult = [];
        Object.keys(resultJson).forEach((type) => {
          resultJson[type]["selected"] = true;
          Object.keys(resultJson[type]["levels"]).forEach((level) => {
            //console.log("Levels : ",resultJson[type]["levels"][level]);
            let resultLevel = resultJson[type]["levels"][level];
            let levelSelectPresent = false;
            if(!this.isLevelSelected(level)) {
              levelSelectPresent = true;
            }
            else {
              levelSelectPresent = resultLevel.some((item) => {
                console.log("Level Select : ",`${level},${item}`);
                let levelSelect = `${level},${item}`;
                return this.selectedRecommenders.some((recommender) => {
                  return recommender == levelSelect;
                });
              });
            }
            console.log("Level select present : ",levelSelectPresent," : ",level);
            if(!levelSelectPresent) {
              resultJson[type]["selected"] = levelSelectPresent;
            }
          });
        });
        Object.keys(resultJson).forEach((type) => {
          if(resultJson[type]["selected"]) {
            this.recommenderResult.push(type);
          }
        })

        this.isCalculated = true;
        console.log("Final Result : ",resultJson);
        //Object.keys;
        //Object.keys(this.selectedRecommenders).forEach();
      },
      isLevelSelected(level) {
        let isLevelSelected = this.selectedRecommenders.some((selectedRecommender) => {
          return selectedRecommender.indexOf(level) > -1;
        });
        return isLevelSelected;
      }
    },
    computed: {
      currentRecommender() {
          return this.recommenders.levels[this.currentLevel];
      },
      isSubmitActive() {
        if(this.selectedRecommenders) {
          return Object.keys(this.selectedRecommenders).length>0;
        }
      }
    }
  }
</script>

<style scoped>

  .recommendation-container {
    position: relative;
  }

  .recommender-header {
    font-size: 30px;
    font-style: normal;
    color: #000;
    font-weight: 600;
  }
  .btn-start {
    font-size: 20px;
    font-weight: 500;
    background: green;
    color: white;
    width: 30%;
    -webkit-border-radius: 7px;
    -moz-border-radius: 7px;
    border-radius: 7px;
    padding: 10px;
    margin-top: 20px;
  }

  .prev {
    float: left;
  }

  .next {
    float: right;
  }

  .prev img, .next img {
    object-fit: cover;
    width: 60px;
    height: 60px;
  }

  .carousel-container {
    height: 100%;
    position: relative;
  }

  .carousel-container .recommender-name {
    padding: 20px;
    text-align: center;
  }
</style>