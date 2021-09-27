<style>
svg {
  padding: 50px;
}
</style>

<script lang="ts">
import { Decimal } from "decimal.js";

let inputQ1: number = 1;
let inputQ2: number = 1;
let inputQ3: number = -1;

let forceMagnitude: string = "";

/**
 * Calculate force magnitude between 2 charges
 */
const calculateForceMagnitude = (
  q1: number,
  q2: number,
  r12: number
): Decimal => {
  return coulombConstant().times(new Decimal((q1 * q2) / (r12 * r12)));
};

/**
 * Calculate Coulomb constant
 *
 * @see https://en.wikipedia.org/wiki/Coulomb_constant
 * @see https://en.wikipedia.org/wiki/Vacuum_permittivity
 *
 * @returns Coulomb constant
 */
const coulombConstant = (): Decimal => {
  return new Decimal(1)
    .times(new Decimal("22468879468420441"))
    .dividedBy(4 * 625000);
};

/**
 *
 * @param input_value
 */
const onInputSet = (input_value: string) => {
  if (!input_value) {
    return;
  }

  forceMagnitude = calculateForceMagnitude(inputQ1, inputQ2, inputQ3)
    .abs()
    .toString();

  forceMagnitude = Number(forceMagnitude).toLocaleString();

  console.log(forceMagnitude);
};

// console.log(coulombConstant().toString());
// console.log(calculateForceMagnitude(1, 3, 4).toString());
</script>

<main>
  <div class="set_input_section">
    <label for="setA">Charge 1 (q1): </label>
    <input
      id="setA"
      name="setA"
      type="number"
      placeholder="1"
      bind:value="{inputQ1}"
      on:input="{onInputSet(inputQ1)}" />
  </div>

  <div class="set_input_section">
    <label for="setB">Charge 2 (q2): </label>
    <input
      id="setB"
      name="setB"
      type="number"
      placeholder="1"
      bind:value="{inputQ2}"
      on:input="{onInputSet(inputQ2)}" />
  </div>

  <div class="set_input_section">
    <label for="charge3">Charge 3 (q3): </label>
    <input
      id="charge3"
      name="charge3"
      type="number"
      placeholder="-1"
      bind:value="{inputQ3}"
      on:input="{onInputSet(inputQ3)}" />
  </div>

  <div class="result">
    |F| = {forceMagnitude}
  </div>

  <div class="drawing">
    <svg
      width="100%"
      height="500"
      version="1.1"
      xmlns="http://www.w3.org/2000/svg">
      <defs>
        <marker
          id="arrowhead"
          markerWidth="4"
          markerHeight="4"
          orient="auto"
          refY="2">
          <path d="M0,0 L4,2 0,4"></path>
        </marker>
      </defs>

      <circle
        cx="25"
        cy="75"
        r="20"
        stroke="red"
        fill="transparent"
        stroke-width="5"></circle>

      <line x1="10" x2="50" y1="110" y2="150" stroke="orange" stroke-width="5"
      ></line>

      <line
        x1="0"
        y1="0"
        x2="50"
        y2="50"
        stroke="#000"
        stroke-width="6"
        marker-end="url(#arrowhead)"></line>
    </svg>
  </div>

  <!-- <canvas bind:this="{canvas}" width="{32}" height="{32}"></canvas> -->
</main>
