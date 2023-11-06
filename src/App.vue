<script setup>
import "bootstrap/dist/css/bootstrap.min.css"
import "bootstrap"

import { reactive, onBeforeMount, computed } from "vue"

const state = reactive({
  input: "",
  output: "",
  binToDec: true,
  inputInvalid: false,
  inputTooBig: false,
  explain: false,
  explainText: "",
})

state.output = computed(() => {
  if (state.binToDec) {
    return binToDec()
  } else {
    return decToBin()
  }
})

/**
 * Two's complement to decimal
 */
function binToDec() {
  if (state.input.trim().length == 0) {
    return ""
  }

  //check for valid chars (0, 1 and " ")
  if (!/^[01 ]+$/.test(state.input)) {
    state.inputInvalid = true
    state.inputTooBig = false
    console.log("invalid chars")
    return ""
  }

  state.inputInvalid = false

  //remove spaces
  let input = state.input.replace(/ /g, "")

  let negative = false
  if (input.length % 8 == 0 && input[0] == 1) {
    negative = true
  }

  if (negative) {
    // invert bits
    input = input.replace(/0/g, "2")
    input = input.replace(/1/g, "0")
    input = input.replace(/2/g, "1")
  }

  let decimal = parseInt(input, 2)

  if (negative) {
    decimal = 0 - (decimal + 1)
  }

  return decimal
}


</script>

<template>
  <h1 class="mb-4">Zweierkomplement Rechner</h1>
  <div class="row">
    <div class="ioBox col-md-12 col-lg-5 d-flex flex-column">
      <div class="labelInputOutput">Eingabe</div>
      <div class="d-flex flex-row align-items-center">
        <div v-if="state.binToDec" class="inputOutputBase">
          Binär:
        </div>
        <div v-else class="inputOutputBase">
          Dezimal:
        </div>

        <div class="ms-2 flex-fill d-flex flex-column">
          <div class="form-floating">
            <input type="text" class="form-control input-lg" :class="{ redBorder: state.inputInvalid }" id="input"
              placeholder="Eingabe" autofocus v-model="state.input">
            <label for="input">Eingabe</label>
          </div>
          <!-- <div class="inputInvalidMessage" :class="{ invisible: !state.inputInvalid }">
            <div v-if="state.inputTooBig">Zahl zu groß (Überlauf)</div>
            <div v-else>Eingabe fehlerhaft!</div>
          </div>  -->
        </div>
      </div>
      <div v-if="state.binToDec" class="explainBinInput">
        <ul>
          <li>
            Trage eine Binärzahl in Zweierkomplementdarstellung ein.
          </li>
          <li>
            Wenn du eine negative Zahl darstellen möchtest, so muss deren Länge ein Vielfaches von 8 sein.
          </li>
        </ul>
      </div>
    </div>
    <div class="col-md-12 col-lg-2 d-flex align-items-center justify-content-center">
      <button type="button" class="btn btn-primary my-3 d-flex align-items-center"
        @click.prevent="state.binToDec = !state.binToDec">
        <div class="me-2">Vertauschen</div>
        <img src="./assets/arrow-right-arrow-left-solid.svg" width="30" height="30">
      </button>
    </div>
    <div class="ioBox col-sm-12 col-lg-5 d-flex flex-column">
      <div class="labelInputOutput">Ausgabe</div>
      <div class="d-flex flex-row">
        <div v-if="!state.binToDec" class="inputOutputBase">
          Binär:
        </div>
        <div v-else class="inputOutputBase">
          Dezimal:
        </div>
        <div class="ms-2 flex-fill d-flex flex-column">
          <textarea class="form-control output" :value="state.output" cols="30" rows="2" readonly></textarea>
        </div>
      </div>
    </div>
  </div>
  <div class="row mt-5 ms-3 mb-2">
    Erklärung:
  </div>
  <div class="row">
    <div class="col">
      <textarea class="form-control explainTextarea" :value="state.explainText" cols="30" rows="15" readonly></textarea>
    </div>
  </div>
</template>

<style scoped>
.input-lg {
  height: 75px;
}

.explainBinInput {
  text-align: left;
  margin-top: 10px;
}

.inputOutputBase {
  font-size: 1.5rem;
  font-weight: bold;
  align-self: center;
}

.explainTextarea {
  font-family: monospace;
  white-space: pre;
  background-color: #f0f0f0;
}

.invisible {
  visibility: hidden;
}

.redBorder {
  border-color: #842029;
  border-width: 2px;
}

.inputInvalidMessage {
  align-self: flex-start;
  background-color: #f8d7da;
  color: #842029;
  padding: 5px;
  border-radius: 5px;
}

.inputInvalid {
  border-color: #842029;
}

input,
select {
  border-color: gray;
}

input {
  font-size: 1.5rem;
}

.output {
  font-size: 1.5rem;
}

.ioBox {
  background-color: rgb(228, 228, 228);
  border-radius: 15px;
  padding: 10px;
}

.labelInputOutput {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 10px;
}
</style>
