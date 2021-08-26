<template>
  <div id="app">
    <div class="container mt-5">
      <div class="row justify-content-between border border-dark mx-1 mx-sm-0">
        <h1 class="text-center border-bottom border-dark py-2 px-3 m-0">
          Simulateur
        </h1>
        <div class="col-lg-5 col-xl-4 py-3 px-5">
          <form>
            <fieldset class="mb-4">
              <label for="type" class="d-block mb-2">Type de prêt</label>
              <select
                id="type"
                class="d-block w-100 border border-dark py-1"
                v-model="selected"
                @change="typeChange"
              >
                <option
                  v-for="(data, index) in credits"
                  :key="index"
                  :value="data"
                >
                  <span v-if="data.description">{{
                    data.description.title
                  }}</span>
                  <span v-else-if="data.i18n">{{ data.i18n.title }}</span>
                </option>
              </select>
            </fieldset>
            <fieldset class="mb-4">
              <label for="montant" class="d-block mb-2">Montant</label>
              <div class="d-flex">
                <input
                  type="number"
                  id="montant"
                  v-model="amount"
                  :min="amount_min"
                  :max="amount_max"
                  :step="increment"
                  class="w-100 border border-dark"
                  @change="
                    checkAmount();
                    checkIncrement();
                  "
                  @keydown="invalidChars"
                />
                <span
                  class="
                    d-flex
                    justify-content-center
                    align-items-center
                    text-center
                    border border-dark
                  "
                >
                  €
                </span>
              </div>
            </fieldset>
            <fieldset class="radio">
              <p v-if="selected">Durée</p>
              <div
                v-if="
                  this.selected.i18n &&
                  this.selected.i18n.title === 'Crédit hypothécaire'
                "
                class="row"
              >
                <div
                  v-for="r in this.selected.ranges"
                  :key="r.range_rate"
                  class="col-4"
                >
                  <div v-for="rd in r.range_duration" :key="rd">
                    <input
                      type="radio"
                      name="time"
                      :id="rd"
                      class="m-0"
                      :value="durationType === 'mois' ? rd : rd * 12"
                      :data-rate="r.range_rate"
                      :data-mensuality="
                        Math.round(
                          [(amount * (r.range_rate / 100)) / 12] /
                            [
                              1 -
                                Math.pow(1 + r.range_rate / 100 / 12, -rd * 12),
                            ]
                        )
                      "
                      :checked="rd == durationDefault"
                      @change="checkDuration"
                    />
                    <label
                      :for="rd"
                      class="border border-dark p-2 text-center w-100 mb-3"
                    >
                      <p class="m-0">{{ rd }} {{ durationType }}</p>
                      <p class="m-0">
                        {{
                          Math.round(
                            [(amount * (r.range_rate / 100)) / 12] /
                              [
                                1 -
                                  Math.pow(
                                    1 + r.range_rate / 100 / 12,
                                    -rd * 12
                                  ),
                              ]
                          )
                        }}€
                      </p>
                    </label>
                  </div>
                </div>
              </div>
              <div v-else class="row">
                <div
                  v-for="d in durationRange"
                  :key="d"
                  class="col-6 col-sm-3 col-lg-4"
                >
                  <input
                    type="radio"
                    name="time"
                    :id="d"
                    class="m-0"
                    :value="durationType === 'mois' ? d : d * 12"
                    :data-mensuality="
                      Math.round(
                        [(amount * (taeg / 100)) / 12] /
                          [1 - Math.pow(1 + taeg / 100 / 12, -d)]
                      )
                    "
                    :checked="d == durationDefault"
                    @change="checkDuration"
                  />
                  <label
                    :for="d"
                    class="border border-dark p-2 text-center w-100 mb-3"
                  >
                    <p class="m-0">{{ d }} {{ durationType }}</p>
                    <p class="m-0">
                      {{
                        Math.round(
                          [(amount * (taeg / 100)) / 12] /
                            [1 - Math.pow(1 + taeg / 100 / 12, -d)]
                        )
                      }}€
                    </p>
                  </label>
                </div>
              </div>
            </fieldset>
          </form>
        </div>
        <div class="col-lg-5 col-xl-4 py-3 px-5">
          <div class="result" v-if="amount && taeg && mensuality && duration">
            <p class="mb-2">Mensualités: {{ mensuality }}€</p>
            <p class="mb-2" v-if="selected.description">
              Type de crédit: {{ selected.description.title }}
            </p>
            <p class="mb-2" v-else-if="selected.i18n">
              Type de crédit: {{ selected.i18n.title }}
            </p>
            <p class="mb-2">Montant: {{ amount }}€</p>
            <p class="mb-2">TAEG: {{ taeg }}%</p>
            <p class="mb-2">Durée: {{ duration }} {{ durationType }}</p>
            <p class="mb-2">Total à payer: {{ total }}€</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import credits from "@/json/credits.json";
export default {
  name: "App",
  data() {
    return {
      credits: credits,
      selected: "",
      amount: "",
      amount_min: "",
      amount_max: "",
      durationRange: "",
      durationType: "",
      durationDefault: "",
      duration: "",
      taeg: "",
      mensuality: "",
      total: "",
      increment: "",
    };
  },
  methods: {
    typeChange() {
      this.amount = this.selected.credit.amount_default;
      this.amount_min = this.selected.credit.amount_min;
      this.amount_max = this.selected.credit.amount_max;
      this.durationDefault = this.selected.credit.range_duration_default;
      this.duration = this.durationDefault;

      if (this.selected.credit.range_duration_type === "months") {
        this.durationType = "mois";
      } else if (this.selected.credit.range_duration_type === "years") {
        this.durationType = "ans";
      }

      if (this.selected.credit.slider_increments) {
        this.increment = this.selected.credit.slider_increments;
      } else {
        this.increment = "";
      }

      this.checkAmount();
    },
    checkAmount() {
      this.checkMinMax();

      if (
        this.selected.i18n &&
        this.selected.i18n.title === "Crédit hypothécaire"
      ) {
        this.taeg = 2.25;
        this.mensuality = Math.round(
          [(this.amount * (this.taeg / 100)) / 12] /
            [1 - Math.pow(1 + this.taeg / 100 / 12, -this.duration * 12)]
        );
      } else {
        for (let r in this.selected.ranges) {
          let min = this.selected.ranges[r].range_min;
          let max = this.selected.ranges[r].range_max;

          if (min <= this.amount && this.amount <= max) {
            this.durationRange = this.selected.ranges[r].range_duration;
            this.taeg = this.selected.ranges[r].range_rate;
            this.mensuality = Math.round(
              [(this.amount * (this.taeg / 100)) / 12] /
                [1 - Math.pow(1 + this.taeg / 100 / 12, -this.duration)]
            );
          }
        }
      }

      let included = 0;
      for (let i in this.durationRange) {
        if (this.durationRange[i] != this.duration) {
          included += 1;
          if (included === this.durationRange.length) {
            this.duration = "";
          }
        }
      }

      this.calcTotal();
    },
    checkIncrement() {
      if (this.increment !== "") {
        while (this.amount % this.increment !== 0) {
          this.amount++;
        }
      }
    },
    invalidChars(e) {
      if (e.key === "E" || e.key === "e" || e.key === "+" || e.key === "-") {
        e.preventDefault();
      }
    },
    checkDuration() {
      if (this.durationType === "mois") {
        this.duration = document.querySelector("input:checked").value;
      } else if (this.durationType === "ans") {
        this.duration = document.querySelector("input:checked").value / 12;
      }
      this.mensuality = document
        .querySelector("input:checked")
        .getAttribute("data-mensuality");

      if (
        this.selected.i18n &&
        this.selected.i18n.title === "Crédit hypothécaire"
      ) {
        this.taeg = document
          .querySelector("input:checked")
          .getAttribute("data-rate");
      }

      this.calcTotal();
    },
    checkMinMax() {
      if (this.amount < this.amount_min) {
        this.amount = this.amount_min;
      } else if (this.amount > this.amount_max) {
        this.amount = this.amount_max;
        this.duration = this.durationDefault;
      }
    },
    calcTotal() {
      if (
        this.selected.i18n &&
        this.selected.i18n.title === "Crédit hypothécaire"
      ) {
        this.total = this.mensuality * this.duration * 12;
      } else {
        this.total = this.mensuality * this.duration;
      }
    },
  },
};
</script>

<style lang="scss">
#app {
  form {
    font-size: 0.875rem;
    span {
      width: 30px;
      height: 30px;
    }
    input {
      width: 0;
      &::-webkit-outer-spin-button,
      &::-webkit-inner-spin-button {
        -webkit-appearance: none;
        margin: 0;
      }
      &[type="number"] {
        -moz-appearance: textfield;
      }
      &:checked {
        & + label {
          background-color: dodgerBlue;
        }
      }
    }
  }
  .result {
    font-size: 1.2em;
  }
}
</style>
