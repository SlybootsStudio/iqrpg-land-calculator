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
        </div>
      </div>
      <div class="alert alert-secondary mb-3 pb-1">
        <div class="fw-bold mb-3">Land Selection</div>
        <div class="row mb-0 pb-0">
          <div class="col-12 col-sm-12 col-md-6 col-lg-4 mb-3">
            <div class="form-floating">
              <select
                class="form-select form-select-lg text-capitalize"
                v-model="landCurrent"
              >
                <option
                  v-for="land in landData"
                  :key="land.level"
                  v-bind:value="land.level"
                >
                  <span class="text-capitalize"
                    >{{ land.name }} ({{ land.level }})</span
                  >
                </option>
              </select>
              <label class="text-light fw-bold">Current Land</label>
            </div>
          </div>
          <div class="col-12 col-sm-12 col-md-6 col-lg-4 mb-3">
            <div class="form-floating">
              <select
                class="form-select form-select-lg text-capitalize"
                v-model="landTarget"
              >
                <option
                  v-for="land in landData"
                  :key="land.level"
                  v-bind:value="land.level"
                >
                  <span class="text-capitalize"
                    >{{ land.name }} ({{ land.level }})</span
                  >
                </option>
              </select>
              <label class="text-light fw-bold">Target Land</label>
            </div>
          </div>
        </div>
        <div class="fw-bold" v-if="warning">
          {{ warning }}
        </div>
      </div>
    </div>
    <div class="col">
      <div class="alert alert-secondary mb-3 overflow-hidden">
        <div class="fs-5 fw-bold">Total Cost</div>

        <p>All costs are in gold.</p>

        <div
          class="d-flex justify-content-between border-bottom border-dark mb-2"
        >
          <div>Metal Cost</div>
          <div>{{ totalCost.metalCost.toLocaleString() }}</div>
        </div>
        <div
          class="d-flex justify-content-between border-bottom border-dark mb-2"
        >
          <div>Wood Cost</div>
          <div>{{ totalCost.woodCost.toLocaleString() }}</div>
        </div>
        <div
          class="d-flex justify-content-between border-bottom border-dark mb-2"
        >
          <div>Stone Cost</div>
          <div>{{ totalCost.stoneCost.toLocaleString() }}</div>
        </div>
        <div
          class="d-flex justify-content-between border-bottom border-dark mb-2"
        >
          <div>Gold</div>
          <div>{{ totalCost.gold.toLocaleString() }}</div>
        </div>
        <div
          class="d-flex justify-content-between fw-bold border-bottom border-dark mb-2"
        >
          <div>Grand Total</div>
          <div>{{ totalCost.total.toLocaleString() }}</div>
        </div>
        <br /><br />
        You would need
        <b>{{ totalCost.resource.toLocaleString() }}</b> of each resource.
      </div>
    </div>
  </div>
  <div class="mb-5"></div>
</template>

<script>
import FloatInput from "@/components/FloatInput";

import landData from "@/data/land.json";
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
      landData: landData, // Static
      metal: 4, // Market Input
      wood: 4, // Market Input
      stone: 4, // Market Input,
      landCurrent: 0, // user selected current land
      landTarget: 1, // user selected target land
      warning: "" // warning message
    };
  },
  computed: {
    totalCost() {
      let currentTotals = this.totalByLevel(this.landCurrent);
      let targetTotals = this.totalByLevel(this.landTarget);

      return {
        resource: targetTotals.resource - currentTotals.resource,
        metalCost: targetTotals.metalCost - currentTotals.metalCost,
        woodCost: targetTotals.woodCost - currentTotals.woodCost,
        stoneCost: targetTotals.stoneCost - currentTotals.stoneCost,
        gold: targetTotals.gold - currentTotals.gold,
        total: targetTotals.total - currentTotals.total
      };
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
    },
    totalByLevel(level) {
      let startingTotal = { resource: 0, gold: 0 };

      // Grab all land entries from 0 to the target.
      let landSlice = this.landData.slice(0, level + 1);
      // Accumulate all resources
      let accumulatedTotal = landSlice.reduce((acc, land) => {
        acc.gold += land.gold;
        acc.resource += land.resource;

        return acc;
      }, startingTotal);

      let gold = accumulatedTotal.gold;

      // multiply by market values
      let metalCost = this.metal * accumulatedTotal.resource;
      let woodCost = this.wood * accumulatedTotal.resource;
      let stoneCost = this.stone * accumulatedTotal.resource;

      return {
        resource: accumulatedTotal.resource,
        metalCost,
        woodCost, // gold cost
        stoneCost,
        gold,
        total: metalCost + woodCost + stoneCost + gold
      };
    },
    checkLandValues() {
      if (this.landCurrent >= this.landTarget) {
        // Target should be 1 above current if it is ever below.
        this.landTarget = this.landCurrent + 1;
        // Cap target to max level
        if (this.landTarget > this.landData.length - 1) {
          this.landTarget = this.landData.length - 1;
        }
      }
    }
  },
  watch: {
    landCurrent() {
      this.warning = "";
      this.checkLandValues();
    },
    landTarget(newVal) {
      if (newVal <= this.landCurrent) {
        this.warning = "Target land must be above current land.";
      } else {
        // this.warning = "";
      }
      this.checkLandValues();
    }
  }
};
</script>
