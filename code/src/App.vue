<script setup>
import { ref, reactive, watch } from "vue";

const isSubmitted = ref(false);
const isValidated = ref(false);

const calculatorData = reactive({
  amount: null,
  term: null,
  rate: null,
  type: null,
  monthlyRepayments: null,
  totalRepayments: null,
});

const errors = reactive({
  amount: "",
  term: "",
  rate: "",
  type: "",
  monthlyRepayments: "",
  totalRepayments: "",
});

function handleAmountInput(event) {
  const input = event.target;
  const rawValue = input.value.replace(/[^0-9.]/g, "");

  if (rawValue.includes(".")) {
    calculatorData.amount = parseFloat(rawValue) || null;
  } else {
    calculatorData.amount = parseInt(rawValue) || null;
  }

  const formattedValue = rawValue.replace(/\B(?=(\d{3})+(?!\d))/g, ",");
  input.value = formattedValue;
}

function handleTermInput(event) {
  const input = event.target;
  const rawValue = input.value.replace(/[^0-9]/g, "");
  calculatorData.term = rawValue ? parseInt(rawValue, 10) : null;
  input.value = rawValue;
}

function calculateMortgage() {
  const principal = calculatorData.amount;
  const annualRate = calculatorData.rate / 100;
  const monthlyRate = annualRate / 12;
  const totalPayments = calculatorData.term * 12;

  if (calculatorData.type === "Repayment") {
    const numerator = monthlyRate * Math.pow(1 + monthlyRate, totalPayments);
    const denominator = Math.pow(1 + monthlyRate, totalPayments) - 1;
    calculatorData.monthlyRepayments = (principal * numerator) / denominator;
  } else if (calculatorData.type === "Interest Only") {
    calculatorData.monthlyRepayments = principal * monthlyRate;
  }

  calculatorData.totalRepayments =
    calculatorData.monthlyRepayments * totalPayments;

  calculatorData.monthlyRepayments = parseFloat(
    calculatorData.monthlyRepayments.toFixed(2)
  ).toLocaleString("en-GB", { minimumFractionDigits: 2 });
  calculatorData.totalRepayments = parseFloat(
    calculatorData.totalRepayments.toFixed(2)
  ).toLocaleString("en-GB", { minimumFractionDigits: 2 });
}

function validateForm() {
  Object.keys(errors).forEach((value) => {
    errors[value] = "";
  });

  if (!calculatorData.amount) {
    errors.amount = "This field is required";
  }

  if (!calculatorData.term) {
    errors.term = "This field is required";
  }

  if (!calculatorData.rate) {
    errors.rate = "This field is required";
  }

  if (!calculatorData.type) {
    errors.type = "This field is required";
  }

  return Object.values(errors).every((error) => error === "");
}

function submitForm() {
  isSubmitted.value = true;

  if (!validateForm()) {
    return;
  }

  calculateMortgage();
  isValidated.value = true;
}

function clearAll() {
  Object.keys(calculatorData).forEach((data) => (calculatorData[data] = null));
  const amountInput = document.getElementById("amount");
  if (amountInput) amountInput.value = "";
  isSubmitted.value = false;
  isValidated.value = false;
}

watch(
  () => calculatorData,
  (newValue) => {
    Object.keys(errors).forEach((key) => {
      if (newValue[key]) {
        errors[key] = "";
      }
    });
  },
  { deep: true }
);
</script>

<template>
  <main>
    <div class="calculator">
      <form @submit.prevent="submitForm">
        <div class="top">
          <h1>Mortgage Calculator</h1>
          <p v-on:click="clearAll">Clear All</p>
        </div>

        <label for="amount">
          Mortgage Amount
          <div
            class="amount-box"
            :class="{ 'error-box': isSubmitted && errors.amount }"
          >
            <span
              class="prefix"
              :class="{ 'error-prefix-sufix': isSubmitted && errors.amount }"
              >£</span
            >
            <input id="amount" type="text" @input="handleAmountInput" />
          </div>
          <span class="error" v-if="isSubmitted && errors.amount">
            {{ errors.amount }}
          </span>
        </label>

        <div class="side">
          <label for="term">
            Mortgage Term
            <div
              class="term-box"
              :class="{ 'error-box': isSubmitted && errors.term }"
            >
              <input
                id="term"
                type="number"
                v-model="calculatorData.term"
                @input="handleTermInput"
              />
              <span
                class="sufix"
                :class="{ 'error-prefix-sufix': isSubmitted && errors.term }"
                >years</span
              >
            </div>
            <span class="error" v-if="isSubmitted && errors.term">
              {{ errors.term }}
            </span>
          </label>

          <label for="rate">
            Interest Rate
            <div
              class="rate-box"
              :class="{ 'error-box': isSubmitted && errors.rate }"
            >
              <input
                id="rate"
                type="number"
                step="any"
                v-model="calculatorData.rate"
              />
              <span
                class="sufix"
                :class="{ 'error-prefix-sufix': isSubmitted && errors.rate }"
                >%</span
              >
            </div>
            <span class="error" v-if="isSubmitted && errors.rate">
              {{ errors.rate }}
            </span>
          </label>
        </div>

        <div class="radio-group">
          <label>Mortgage Type</label>
          <label
            for="repayment"
            class="radio-container"
            :class="{ selected: calculatorData.type === 'Repayment' }"
          >
            <input
              class="radio-input"
              id="repayment"
              type="radio"
              name="typeRadio"
              value="Repayment"
              v-model="calculatorData.type"
            />
            Repayment
          </label>

          <label
            for="interest"
            class="radio-container"
            :class="{ selected: calculatorData.type === 'Interest Only' }"
          >
            <input
              class="radio-input"
              id="interest"
              type="radio"
              name="typeRadio"
              value="Interest Only"
              v-model="calculatorData.type"
            />
            Interest Only
          </label>
          <span class="error-radio" v-if="isSubmitted && errors.type">
            {{ errors.type }}
          </span>
        </div>

        <div class="submit-button">
          <img
            src="./assets/images/icon-calculator.svg"
            alt="Icon Calculator"
          />
          <input class="submit" type="submit" value="Calculate Repayments" />
        </div>
      </form>

      <section class="no-results" v-if="!isValidated">
        <img
          src="./assets/images/illustration-empty.svg"
          alt="Illustration Empty"
        />
        <h1>Results shown here</h1>

        <p>
          Complete the form and click “calculate repayments” to see what your
          monthly repayments would be.
        </p>
      </section>

      <section class="results" v-if="isValidated">
        <div class="results-intro">
          <h1>Your results</h1>
          <p>
            Your results are shown below based on the information you provided.
            To adjust the results, edit the form and click “calculate
            repayments” again.
          </p>
        </div>

        <div class="result-box">
          <div class="monthlyRepay">
            <p>Your monthly repayments</p>
            <h2>£{{ calculatorData.monthlyRepayments }}</h2>
          </div>
          <div class="totalRepay">
            <p>Total you'll repay over the term</p>
            <h3>£{{ calculatorData.totalRepayments }}</h3>
          </div>
        </div>
      </section>
    </div>
  </main>
</template>

<style>
html,
body {
  background-color: hsl(202, 86%, 94%);
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
}

@font-face {
  font-family: "Plus Jakarta Sans";
  src: url("./assets/fonts/static/PlusJakartaSans-Medium.ttf")
    format("truetype");
  font-style: normal;
  font-weight: 500;
}

@font-face {
  font-family: "Plus Jakarta Sans";
  src: url("./assets/fonts/static/PlusJakartaSans-Bold.ttf") format("truetype");
  font-style: normal;
  font-weight: 700;
}

* {
  font-family: "Plus Jakarta Sans", sans-serif;
  font-size: 1rem;
}
</style>

<style scoped>
main {
  display: flex;
  flex-direction: row;
  height: 100dvh;
  width: 100dvw;
  justify-content: center;
  align-items: center;
}

.calculator {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: stretch;
}

form {
  display: flex;
  flex-direction: column;
  justify-content: center;
  width: max(400px, calc(37% + 1vw));
  max-width: 40%;
  background-color: hsl(0, 0%, 100%);
  border-radius: 20px 20px 0 20px;
  resize: none;
  box-sizing: border-box;
  padding: 1% 10% 0 2%;
  position: relative;
  left: 3vw;
}

.top {
  display: flex;
  flex-direction: row;
  align-items: top;
  justify-content: space-between;
  margin-bottom: 35px;
  position: relative;
  top: 1vh;
}

.top h1 {
  font-size: 1.5rem;
  margin-top: 10px;
}

.top p {
  align-self: center;
  text-decoration: underline;
  color: hsl(200, 24%, 40%);
  cursor: pointer;
  margin-top: 10px;
}

.top p:hover {
  color: hsl(202, 55%, 16%);
}

label {
  color: hsl(200, 24%, 40%);
  width: 100%;
}

input {
  font-weight: 700;
  width: 100%;
  height: 90%;
}

.amount-box {
  display: flex;
  align-items: center;
  height: 45px;
  width: 100%;
  border: 1px solid #a0a0a0;
  border-radius: 5px;
  margin: 12px 0 25px;
  resize: none;
  box-sizing: border-box;
}

.amount-box:hover {
  cursor: pointer;
  border-color: hsl(202, 55%, 16%);
}

.amount-box input {
  border: none;
  outline: none;
  padding-left: 15px;
  cursor: pointer;
}

.amount-box .prefix {
  background-color: hsl(202, 86%, 94%);
  border-top-left-radius: 5px;
  border-bottom-left-radius: 5px;
  padding: 0 15px;
  height: 100%;
  align-content: center;
}

.amount-box:focus-within {
  border-color: hsl(61, 70%, 52%);
}

.amount-box:focus-within .prefix {
  background-color: hsl(61, 70%, 52%);
  color: black;
}

.side {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  width: 100%;
  gap: 20px;
}

.side label {
  display: flex;
  flex-direction: column;
  flex: 1;
}

.term-box,
.rate-box {
  display: flex;
  align-items: center;
  border: 1px solid #a0a0a0;
  border-radius: 5px;
  height: 45px;
  width: 100%;
  margin: 5px 0 25px;
  resize: none;
  box-sizing: border-box;
}

.term-box:hover,
.rate-box:hover {
  cursor: pointer;
  border-color: hsl(202, 55%, 16%);
}

.term-box:focus-within,
.rate-box:focus-within {
  border-color: hsl(61, 70%, 52%);
}

.term-box input,
.rate-box input {
  border: none;
  outline: none;
  padding: 0 15px;
  width: 100%;
  cursor: pointer;
}

.term-box .sufix,
.rate-box .sufix {
  background-color: hsl(202, 86%, 94%);
  border-top-right-radius: 5px;
  border-bottom-right-radius: 5px;
  padding: 0 15px;
  height: 100%;
  align-content: center;
}

.term-box:focus-within .sufix,
.rate-box:focus-within .sufix {
  background-color: hsl(61, 70%, 52%);
  color: black;
}

.radio-group {
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 10px;
}

.radio-container {
  display: flex;
  align-items: center;
  height: 45px;
  border: 1px solid hsl(202, 55%, 16%);
  border-radius: 5px;
  padding: 0 15px;
  resize: none;
  box-sizing: border-box;
  gap: 10px;
  color: hsl(202, 55%, 16%);
  font-weight: 700;
  cursor: pointer;
  transition: background-color 0.5s;
}

.radio-container:hover {
  border-color: hsl(61, 70%, 52%);
}

.radio-container input.radio {
  width: 20px;
}

.radio-container.selected {
  border-color: hsl(61, 70%, 52%);
}

.radio-container input[type="radio"] {
  appearance: none;
  cursor: pointer;
  width: 14px;
  height: 14px;
  border: 2px solid hsl(202, 55%, 16%);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: white;
}

.radio-container input[type="radio"]:checked {
  background-color: hsl(61, 70%, 52%);
  border: 3px solid white;
  box-shadow: 0 0 0 2px hsl(61, 70%, 52%);
}

.radio-container.selected {
  background-color: hsl(61, 70%, 52%, 15%);
}

.submit-button {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 5vh;
  height: 5vh;
  width: 75%;
  border: none;
  background-color: hsl(61, 70%, 52%);
  border-radius: 50px;
  resize: none;
  box-sizing: border-box;
  gap: 10px;
  margin: 30px 0;
}

.submit-button input {
  border: none;
  outline: none;
  background-color: transparent;
  cursor: pointer;
  width: min-content;
}

.submit-button:hover {
  cursor: pointer;
  background-color: hsl(61, 70%, 52%, 35%);
}

.submit-button:hover input {
  background-color: transparent;
}

section {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin: 0;
  width: max(480px, calc(32% + 1vw));
  max-width: 40%;
  background-color: hsl(202, 55%, 16%);
  border-radius: 0 20px 20px 80px;
  position: relative;
  right: 3vw;
  z-index: 2;
  padding: 30px;
  resize: none;
  box-sizing: border-box;
}

.results-intro {
  margin-bottom: 25px;
}

.results h1 {
  color: hsl(0, 0%, 100%);
  font-size: 1.5rem;
  margin-top: 0;
}

.results p {
  color: hsl(203, 41%, 72%);
}

.result-box {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: hsl(202, 55%, 13%);
  border-top: 5px solid hsl(61, 70%, 52%);
  border-radius: 7px;
  width: 100%;
  margin: 0 auto 5vh;
  resize: none;
  box-sizing: border-box;
  padding: 0 35px;
}

.monthlyRepay,
.totalRepay {
  display: flex;
  flex-direction: column;
  background-color: transparent;
  width: 100%;
  padding: 20px 0;
}

.monthlyRepay p {
  margin: 10px 0;
}

.monthlyRepay h2 {
  color: hsl(61, 70%, 52%);
  font-size: 3.2rem;
  padding: 0;
  margin: 0;
}

.monthlyRepay {
  border-bottom: 1px solid hsl(200, 24%, 40%);
}

.totalRepay h3 {
  color: hsl(0, 0%, 100%);
  font-size: 1.5rem;
  margin: 0;
  margin-bottom: 20px;
}

.totalRepay p {
  margin: 5px 0 7px;
}

.no-results {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.no-results h1 {
  color: hsl(0, 0%, 100%);
}

.no-results p {
  color: hsl(203, 41%, 72%);
  text-align: center;
}

.no-results img {
  max-width: 100%;
  height: auto;
  aspect-ratio: 1/1;
}

input[type="number"] {
  appearance: none;
  -webkit-appearance: none;
  -moz-appearance: textfield;
}

input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  appearance: none;
  margin: 0;
}

.error {
  color: hsl(4, 69%, 50%);
  font-size: 0.8rem;
  margin: -17px 0 20px;
  display: block;
}

.error-radio {
  color: hsl(4, 69%, 50%);
  font-size: 0.8rem;
  margin: 0;
  display: block;
}

.error:empty {
  display: none;
}

.error-box {
  border: 1px solid hsl(4, 69%, 50%) !important;
}

.error-prefix-sufix {
  background-color: hsl(4, 69%, 50%) !important;
  color: hsl(0, 0%, 100%) !important;
}

@media (max-width: 1024px) {
  main {
    flex-direction: column;
    height: auto;
    padding: 0;
  }

  .calculator {
    flex-direction: column;
  }

  form,
  section {
    width: 100%;
    max-width: 100%;
    border-radius: 0;
    margin: 0;
  }

  form {
    position: inherit;
    padding: 30px 25px 0;
    height: auto;
  }

  .top {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.7rem;
    margin-bottom: 1.7rem;
    top: 0;
  }

  .top h1,
  .top p {
    margin: 0;
    align-self: auto;
  }

  .side {
    flex-direction: column;
    gap: 0;
  }

  .radio-container,
  .submit,
  .prefix,
  .sufix {
    font-weight: 700;
  }

  .submit-button {
    width: 100%;
    height: auto;
    padding: 15px;
    flex-direction: row;
  }

  .radio-container {
    height: auto;
    padding: 15px;
  }

  section {
    position: inherit;
    height: auto;
    padding: 15px;
  }

  .result-box {
    height: auto;
    margin-bottom: 10px;
    padding: 15px;
  }

  .results-intro {
    margin-bottom: 10px;
  }

  .monthlyRepay h2 {
    font-size: 2rem;
  }

  .results h1,
  .results h3,
  .no-results h1 {
    font-size: 1.3rem;
  }

  .monthlyRepay {
    padding-top: 0;
  }

  .totalRepay {
    padding: 5px 0;
  }

  .results h3 {
    margin-bottom: 0;
  }

  .results p,
  .no-results p {
    font-size: 0.9rem;
  }
}
</style>
