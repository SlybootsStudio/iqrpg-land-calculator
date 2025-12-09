<template>
  <div class="row mt-3">
    <div class="col-12 col-md-8">
      <div class="alert alert-secondary mb-3 pb-1">
        <div class="fw-bold mb-3">Market Values</div>
        <div class="row mb-0 pb-0">
          <div class="col-12 col-sm-12 col-md-6 col-lg-4 mb-3">
            <FloatInput
              :min="0"
              :value="metal"
              label="Metal"
              @eInput="setMetal($event)"
            />
          </div>
          <div class="col-12 col-sm-12 col-md-6 col-lg-4 mb-3">
            <FloatInput
              :min="0"
              :value="wood"
              label="Wood"
              @eInput="setWood($event)"
            />
          </div>
          <div class="col-12 col-sm-12 col-md-6 col-lg-4 mb-3">
            <FloatInput
              :min="0"
              :max="125"
              :limit="125"
              :value="stone"
              label="Stone"
              @eInput="setStone($event)"
            />
          </div>
          <div class="col-12 col-sm-12 col-md-6 col-lg-4 mb-3">
            <div class="form-floating">
              <select
                class="form-select form-select-lg text-capitalize"
                v-model="land"
              >
                <option
                  v-for="(land, i) in landLevels"
                  :key="i"
                  v-bind:value="i"
                >
                  <span class="text-capitalize">{{ land }} ({{ i }})</span>
                </option>
              </select>
              <label class="text-light fw-bold">Land</label>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="col">
      <div class="alert alert-secondary mb-3 fs-2 overflow-hidden">
        Land Costs....

        <p>{{ metal.toLocaleString() }} Metal</p>
        <p>{{ wood.toLocaleString() }} Wood</p>
        <p>{{ stone.toLocaleString() }} Stone</p>
      </div>
    </div>
  </div>
  <div class="mb-5"></div>
</template>

<script>
import FloatInput from "@/components/FloatInput";

/*
To show action rare, epic, legendary ONLY chance, subtract out the chance for the 'level up'
*/
export default {
  name: "Main",
  components: {
    FloatInput
  },
  data() {
    return {
      metal: 0,
      wood: 0,
      stone: 0,
      land: 0,
      landLevels: [
        "no land",
        "camp",
        "small village",
        "village",
        "large village",
        "small town",
        "town",
        "large town",
        "small city",
        "city",
        "large city",
        "small kingdom",
        "kingdom",
        "large kingdom",
        "small empire",
        "empire",
        "large empire"
      ]
    };
  },
  computed: {
    totalOutput() {
      let total =
        125000 /
        (1 + this.trinketDropChanceTotal / 100) /
        (1 + this.dropBoostTotal / 100);

      total = Math.round(total);
      return total;
    },
    trinketDropChanceTotal() {
      let total = 0;

      total += this.skillLevelTotal * 1; // number. (1-200)
      total += this.land * 2; // number (0-24)
      total += this.treasureHunterLevel * 3; //1-150)

      return total;
    },
    dropBoostTotal() {
      let total = 0;

      total += this.clanDropTotem * 2; // % 20 -> 40%
      total += this.premiumDrop; // % - 0 - 125%
      total += this.trinketDropBoostTotal; // %

      return total;
    },
    skillLevelTotal() {
      /*
      1% <= 100
      2% > 100
*/
      let total = 0;
      total += this.skillLevel;

      if (this.skillLevel > 100) {
        total += this.skillLevel - 100;
      }

      return total;
    },
    trinketDropBoostTotal() {
      let total = 0;

      this.trinkets.map((trinket) => {
        total += trinket.dropBoost;
      });

      return total;
    }
  },
  methods: {
    setMetal(value) {
      this.metal = value;
    },
    setWood(value) {
      this.wood = value;
    },
    setStone(value) {
      this.stone = value;
    }
  }
};
</script>
