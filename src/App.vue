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

state.explainText = computed(() => {
  if (state.input.trim().length == 0) {
    return ""
  }

  if (state.binToDec) {
    return explainTwosToDec()
  } else {
    return explainDecToTwos()
  }
})

function switchBinDec() {
  let temp = state.output
  state.binToDec = !state.binToDec
  state.input = temp.toString()
}

function explainTwosToDec() {
  if (state.inputInvalid) {
    return ""
  }

  let input = state.input.replace(/ /g, "")
  let lengthMultipleOf8 = input.length % 8 == 0
  let negative = lengthMultipleOf8 && input[0] == 1

  let result = ""

  if (negative) {
    result += "Die Eingabe ist negativ, da die Länge der Eingabe ein Vielfaches von 8 ist und da das erste Bit eine 1 ist.\n\n"

    result += "1. Merke dir, dass das Ergebnis eine negative Zahl sein muss.\n"

    result += "2. Invertiere alle Bits.\n"
    input = input.replace(/0/g, "2")
    input = input.replace(/1/g, "0")
    input = input.replace(/2/g, "1")
    result += "   -> " + input + "\n"

    result += "3. Addiere 1.\n"
    let decimal = parseInt(input, 2)
    decimal = decimal + 1
    let binary = decimal.toString(2)
    result += "   -> " + binary.padStart(input.length, "0") + "\n"
    result += "    = " + binary.padStart(input.length, " ") + "\n"

    result += "4. Wandle die Binärzahl wie üblich in eine Dezimalzahl um.\n"
    result += explainBinToDec(binary)
    result += "\n"

    result += "5. Füge nun noch das Minuszeichen aus Schritt 1 an.\n"
    result += "   -> -" + decimal + "\n"

  } else if (lengthMultipleOf8 && !negative) {
    result += "Die Eingabe ist positiv, da die Länge der Eingabe zwar ein Vielfaches von 8 ist, das erste Bit aber eine 0 ist.\n\n"
  } else {
    result += "Die Eingabe ist positiv, da die Länge der Eingabe kein Vielfaches von 8 ist.\n\n"
  }

  if (!negative) {
    result += "Wandle die Binärzahl wie üblich in eine Dezimalzahl um.\n\n"
    result += explainBinToDec(input)
    result += "\n"
  }

  return result
}

function getMaxLength(digit, base, power) {
  let result = (digit * base ** power).toString().length
  let calc1 = `${digit} * ${base}^${power}`.length
  let calc2 = `${digit} * ${base ** power}`.length
  return Math.max(result, calc1, calc2)
}

function explainBinToDec(binary) {
  let result = "     "

  // 2 * 10^4 + 3 * 10^3 + ...
  for (let i = 0; i < binary.length; i++) {
    let digit = parseInt(binary[i], 2)
    let power = binary.length - i - 1
    let maxLength = getMaxLength(digit, 2, power)
    let newSum = `${digit} \u2219 2^${power} + `
    result += newSum.padStart(maxLength + 3, " ")
  }

  result = result.slice(0, -3)
  result += "\n   = "

  // 2 * 1000 + 3 * 100 + 4 * 10 + 5 * 1
  for (let i = 0; i < binary.length; i++) {
    let digit = parseInt(binary[i], 2)
    let power = binary.length - i - 1
    let maxLength = getMaxLength(digit, 2, power)
    let newSum = `${digit} \u2219 ${2 ** power} + `
    result += newSum.padStart(maxLength + 3, " ")
  }

  result = result.slice(0, -3)
  result += "\n   = "

  // 2000 + 300 + 40 + 5
  for (let i = 0; i < binary.length; i++) {
    let digit = parseInt(binary[i], 2)
    let power = binary.length - i - 1
    let maxLength = getMaxLength(digit, 2, power)
    let newSum = `${digit * 2 ** power} + `
    result += newSum.padStart(maxLength + 3, " ")
  }

  result = result.slice(0, -3)
  result += "\n   = "

  let lineLength = result.split("\n")[0].length
  result += " ".repeat(Math.max(lineLength - parseInt(binary, 2).toString().length - 5, 0))

  result += parseInt(binary, 2)

  return result
}


function explainDecToTwos() {
  if (state.inputInvalid || state.input.trim() == "-") {
    return ""
  }

  let input = state.input.trim()
  let result = ""

  if (input[0] != "-") {
    result += "Da die Eingabe positiv ist, kannst du die Zahl einfach wie üblich ins Binärsystem umrechnen.\n\n"
  } else {
    result += "1. Interpretiere die Eingabe als positive Zahl und tue zunächst so, als gäbe es kein Vorzeichen.\n"
    result += "   Merke dir das Vorzeichen für Schritt 2.\n"
    result += "   Wandle die Eingabe zunächst wie üblich ins Dezimalsystem um.\n\n"
  }

  result += explainDecToBin(input[0] == "-" ? input.slice(1) : input)

  result += "\n   Füge nun so viele Nullen vorne an, dass die Länge der Binärzahl ein Vielfaches von 8 ist.\n"
  let binaryVomBetrag = parseInt(input[0] == "-" ? input.slice(1) : input).toString(2)
  binaryVomBetrag = binaryVomBetrag.padStart(Math.ceil(binaryVomBetrag.length / 8) * 8, "0")
  binaryVomBetrag = binaryVomBetrag.replace(/(.{8})/g, "$1 ")
  result += `   Resultat: ${binaryVomBetrag}`

  if (input[0] == "-") {
    result += "\n\n2. Da die Zahl negativ war, musst du nun alle Bits invertieren.\n"
    let binary = binaryVomBetrag.replace(/0/g, "2")
    binary = binary.replace(/1/g, "0")
    binary = binary.replace(/2/g, "1")
    result += "   -> " + binary + "\n"

    result += "3. Addiere 1.\n"
    result += "   -> " + state.output

  }

  return result
}

function explainDecToBin(decimal) {
  let result = ""

  let max1 = decimal.length
  decimal = parseInt(decimal)
  let max2 = Math.floor(decimal / 2).toString().length

  while (decimal !== 0) {
    let rest = decimal % 2
    let newLine = "   "
    newLine += " ".repeat(max1 - decimal.toString().length)
    newLine += decimal.toString()
    newLine += " : 2 = "
    newLine += " ".repeat(max2 - (Math.floor(decimal / 2).toString().length))
    newLine += `${Math.floor(decimal / 2)}`
    newLine += ` Rest ${rest}\n`
    result += newLine
    decimal = Math.floor(decimal / 2)
  }

  return result
}

/**
 * Two's complement to decimal
 */
function binToDec() {
  if (state.input.trim().length == 0) {
    state.inputInvalid = false
    return ""
  }

  //check for valid chars (0, 1 and " ")
  if (!/^[01 ]+$/.test(state.input)) {
    state.inputInvalid = true
    state.inputTooBig = false
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


/**
 * Decimal to two's complement
 */
function decToBin() {
  if (state.input.trim().length == 0 || state.input.trim() == "-") {
    state.inputInvalid = false
    return ""
  }

  //check for valid chars (0-9)
  if (!/^-?[0-9]*$/.test(state.input.trim())) {
    state.inputInvalid = true
    state.inputTooBig = false
    return ""
  }

  state.inputInvalid = false

  let decimal = parseInt(state.input)

  //TODO reverse check
  if (decimal > 2 ** 31 - 1 || decimal < -(2 ** 31)) {
    state.inputInvalid = true
    state.inputTooBig = true
    return ""
  }

  if (decimal >= 0) {
    let binary = decimal.toString(2)

    // pad to next multiple of 8
    binary = binary.padStart(Math.ceil(binary.length / 8) * 8, "0")

    // insert whitespace every 8th char
    binary = binary.replace(/(.{8})/g, "$1 ")
    return binary
  }


  decimal = 0 - decimal
  decimal = decimal - 1
  let binary = decimal.toString(2)
  binary = binary.replace(/0/g, "2")
  binary = binary.replace(/1/g, "0")
  binary = binary.replace(/2/g, "1")

  // pad to next multiple of 8
  binary = binary.padStart(Math.ceil(binary.length / 8) * 8, "1")

  // insert whitespace every 8th char
  binary = binary.replace(/(.{8})/g, "$1 ")

  return binary

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
          <div class="inputInvalidMessage" v-if=state.inputInvalid>
            <div v-if="state.inputTooBig">Zahl zu groß</div>
            <div v-else>Eingabe fehlerhaft!</div>
          </div>
        </div>
      </div>
      <div v-if="state.binToDec" class="explainBinInput">
        <ul>
          <li>
            Trage eine Binärzahl in Zweierkomplementdarstellung ein.
          </li>
          <li>
            Wenn du eine negative Zahl darstellen möchtest, so <u>muss</u> deren Länge ein Vielfaches von 8 sein.
          </li>
        </ul>
      </div>
    </div>
    <div class="col-md-12 col-lg-2 d-flex align-items-center justify-content-center">
      <button type="button" class="btn btn-primary my-3 d-flex align-items-center" @click.prevent="switchBinDec()">
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
  <div class="row mt-5 ms-3 mb-2 explainHeader">
    Erklärung:
  </div>
  <div class="row">
    <div class="col">
      <textarea class="form-control explainTextarea" :value="state.explainText" cols="30" rows="15" readonly></textarea>
    </div>
  </div>
</template>

<style scoped>
.explainHeader {
  font-size: 1.2rem;
}

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
  height: 100%;
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
