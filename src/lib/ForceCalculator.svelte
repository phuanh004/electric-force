<script lang="ts">
import { Decimal } from "decimal.js";

const COORDINATES_SYSTEM_MAX_LENGTH = 10;
const SCALE: number = 40;
let forceScale = 0; // Scale force base on log 2

let q1: Charge, q2: Charge, q3: Charge; // 3 Charges

$: ForcesDraw = {
  F1on2: {
    magnitude: Number(force(q1, q2).F1on2.magnitude).toLocaleString(),
    p1: {},
  },
};

const drawForce = (q1: Charge, q2: Charge) => {
  return {
    F1on2: {
      magnitude: Number(force(q1, q2).F1on2.magnitude).toLocaleString(),
      color: "red",
      p1: {},
    },
  };
};

$: F1on2 = Number(force(q1, q2).F1on2.magnitude).toLocaleString();
$: F2on1 = Number(force(q1, q2).F2on1.magnitude).toLocaleString();

type Point = {
  location: Coordinate;
};

type Line = {
  p1: Coordinate;
  p2: Coordinate;
};

type Charge = Point & {
  value: number;
  // location: Coordinate;
};

type Vector = Line & {
  magnitude: number;
  magnitude_scaled?: number;
  color?: string;
};

/**
 * y = mx + b
 */
type LinearEquation = {
  slope: number;
  y_int: number;
};

type Coordinate = {
  x: number;
  y: number;
};

// let forceMagnitude: string = "";

$: error = "";

/**
 * Calculate force magnitude between 2 charges
 *
 * @param q1  first charge
 * @param q2  second charge
 * @param r12 the distance between 2 charges
 */
const forceMagnitude = (q1: Charge, q2: Charge, r12: number): Decimal => {
  return coulombConstant().times(
    new Decimal(Math.abs(q1.value * q2.value) / (r12 * r12))
  );
};

const force = (
  q1: Charge,
  q2: Charge
): {
  F1on2: Vector;
  F2on1: Vector;
} => {
  const eq = linearEquation(q1, q2);
  const r12 = vectorOfTwoPoint(q1, q2).magnitude;
  const magnitude = forceMagnitude(q1, q2, r12).toNumber();
  const base = Decimal.log2(magnitude);

  forceScale =
    forceScale !== 0 ? Decimal.log2(magnitude).toNumber() : forceScale;

  const magnitude_scaled = Decimal.pow(
    magnitude,
    new Decimal(1).dividedBy(base)
  ).toNumber();

  let F2on1_x1 = isPush(q1, q2)
    ? q1.location.x - magnitude_scaled
    : q1.location.x + magnitude_scaled;
  let F2on1_x2 = eq.slope * F2on1_x1 + eq.y_int;

  let F1on2_x1 = isPush(q1, q2)
    ? q2.location.x + magnitude_scaled
    : q2.location.x - magnitude_scaled;
  let F1on2_x2 = eq.slope * F1on2_x1 + eq.y_int;

  let F2on1: Vector = {
    magnitude: magnitude,
    magnitude_scaled: magnitude_scaled,
    color: q2.value > 0 ? "red" : "blue",
    p1: q1.location,
    p2: {
      x: F2on1_x1,
      y: F2on1_x2,
    },
  };

  let F1on2: Vector = {
    magnitude: magnitude,
    magnitude_scaled: magnitude_scaled,
    color: q1.value > 0 ? "red" : "blue",
    p1: q2.location,
    p2: {
      x: F1on2_x1,
      y: F1on2_x2,
    },
  };

  // console.log({
  //   F1on2,
  //   F2on1,
  //   eq,
  // });

  return {
    F1on2: F1on2,
    F2on1: F2on1,
  };
};

const distanceOfTwoPointCharge = (q1: Charge, q2: Charge): number => {
  return Math.sqrt(
    Math.pow(q1.location.x - q2.location.x, 2) +
      Math.pow(q1.location.y - q2.location.y, 2)
  );
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

const linearEquation = (p1: Point, p2: Point): LinearEquation => {
  const slope =
    (p2.location.y - p1.location.y) / (p2.location.x - p1.location.x);
  return {
    slope: (p2.location.y - p1.location.y) / (p2.location.x - p1.location.x),
    y_int: p1.location.y - slope * p1.location.x,
  };
};

const vectorOfTwoPoint = (p1: Point, p2: Point): Vector => {
  const eq = linearEquation(p1, p2);
  const x1 = p1.location.x;
  const x2 = p1.location.y;
  const magnitude = Math.sqrt(
    Math.pow(p1.location.x - p2.location.x, 2) +
      Math.pow(p1.location.y - p2.location.y, 2)
  );

  return {
    magnitude: magnitude,
    p1: {
      x: x1,
      y: eq.slope * x1 + eq.y_int,
    },
    p2: {
      x: x2,
      y: eq.slope * x2 + eq.y_int,
    },
  };
};

/**
 *
 * @param input_value
 */
const onInputSet = (input_value: number) => {
  if (input_value === -1) {
    console.log("Nope");
    error = "Nope";

    return 0;
  }

  return input_value;
  // if (!input_value && !isDataValid(inputQ1, inputQ2, inputQ3)) {
  //   error = "Charge cannot equal zero";
  //   return;
  // }
  // forceMagnitude = calculateForceMagnitude(inputQ1, inputQ2, inputQ3)
  //   .abs()
  //   .toString();
  // forceMagnitude = Number(forceMagnitude).toLocaleString();
  // console.log(forceMagnitude);
};

/**
 * Check data is valid
 * q1, q2, q3 must be not equal to zero
 * q1 =/= q2 =/= q3
 * Two charge
 *
 * @param q1 charge 1
 * @param q2 charge 2
 * @param q3 charge 3
 */
// const isDataValid = (q1: number, q2: number, q3: number): boolean => {
//   if (q1 === 0 || q2 === 0 || q3 === 0) return false;

//   // TODO: Check 2 charges positive and 1 charge negative

//   return q1 !== q2 && q2 !== q3 && q1 !== q3;
// };

const isValidChargeSystem = (q1: Charge, q2: Charge, q3: Charge): boolean => {
  if (q1.value !== q2.value && q2.value !== q3.value && q1 !== q3) {
    return true;
  } else {
    error = "q1, q2, and q3 could not be the same";
    return false;
  }
};
const onInputSetCharge = (event: InputEvent) => {
  if (!isValidChargeSystem(q1, q2, q3)) return;

  const id: string = event.target?.id;
  const charge: number = event.target?.value;

  switch (id) {
    case "charge1":
      q1.value = charge;
      break;
    case "charge2":
      q1.value = charge;
      break;
    case "charge3":
      q3.value = charge;
      break;
  }
};

const colorChargeCSS = (q: Charge): string => {
  return q.value > 0 ? "charge-red" : "charge-blue";
};

/**
 * Convert cartesian system to SVG system with the scale
 * @param param0
 */
function cartesianToSVG(x: number, y: number) {
  return {
    x: x * SCALE,
    y: y * SCALE,
  };
}

const isPush = (q1: Charge, q2: Charge): boolean => {
  return new Decimal(q1.value * q2.value).isPositive();
};

const initData = () => {
  q1 = {
    value: 0.3,
    location: {
      x: 3,
      y: 8,
    },
  };

  q2 = {
    value: -0.1,
    location: {
      x: 8,
      y: 2,
    },
  };

  q3 = {
    value: 0.2,
    location: {
      x: 13,
      y: 3,
    },
  };
};

initData();
</script>

<main>
  <div class="set_input_section">
    <label for="charge1">Charge 1 (q1): </label>
    <input
      id="charge1"
      name="charge1"
      type="number"
      placeholder="1"
      value="1"
      on:input="{onInputSetCharge}" />
    <!-- bind:value="{q1.value}" -->
  </div>

  <div class="set_input_section">
    <span>Charge 1 location: </span>
    <label for="charge1_location_x">x: </label>
    <input
      id="charge1_location_x"
      name="charge1_location_x"
      type="number"
      placeholder="1"
      bind:value="{q1.location.x}" />

    <label for="charge1_location_y">y: </label>
    <input
      id="charge1_location_y"
      name="charge1_location_y"
      type="number"
      placeholder="1"
      bind:value="{q1.location.y}" />
  </div>

  <div class="set_input_section">
    <label for="charge2">Charge 2 (q2): </label>
    <input
      id="charge2"
      name="charge2"
      type="number"
      placeholder="1"
      bind:value="{q2.value}" />
  </div>

  <div class="set_input_section">
    <span>Charge 2 location: </span>
    <label for="charge2_location_x">x: </label>
    <input
      id="charge2_location_x"
      name="charge2_location_x"
      type="number"
      placeholder="1"
      bind:value="{q2.location.x}" />

    <label for="charge2_location_y">y: </label>
    <input
      id="charge2_location_y"
      name="charge2_location_y"
      type="number"
      placeholder="1"
      bind:value="{q2.location.y}" />
  </div>

  <div class="set_input_section">
    <label for="charge3">Charge 3 (q3): </label>
    <input
      id="charge3"
      name="charge3"
      type="number"
      placeholder="-1"
      bind:value="{q3.value}" />
    <!-- on:input="{onInputSet(inputQ3)}" -->
  </div>

  <div class="set_input_section">
    <span>Charge 3 location: </span>
    <label for="charge3_location_x">x: </label>
    <input
      id="charge3_location_x"
      name="charge3_location_x"
      type="number"
      placeholder="1"
      bind:value="{q3.location.x}" />

    <label for="charge3_location_y">y: </label>
    <input
      id="charge3_location_y"
      name="charge3_location_y"
      type="number"
      placeholder="1"
      bind:value="{q3.location.y}" />

    <div class="error">
      {error}
    </div>

    <div class="result">
      |F12| = {F1on2}
    </div>

    <div class="drawing">
      <svg
        width="100%"
        height="{COORDINATES_SYSTEM_MAX_LENGTH * SCALE}"
        style="transform: rotateX(180deg);"
        version="1.1"
        xmlns="http://www.w3.org/2000/svg">
        <defs>
          <marker
            id="arrow-head-blue"
            markerWidth="4"
            markerHeight="4"
            orient="auto"
            refY="2">
            <path d="M0,0 L4,2 0,4"></path>
          </marker>
        </defs>

        <!-- Coodinate System -->
        <!-- x -->
        <text
          x="98%"
          y="{cartesianToSVG(0, COORDINATES_SYSTEM_MAX_LENGTH - 9.5).y}"
          >x</text>

        <line
          x1="{cartesianToSVG(0, 0).x}"
          y1="{cartesianToSVG(0, 0).y}"
          x2="100%"
          y2="{cartesianToSVG(COORDINATES_SYSTEM_MAX_LENGTH, 0).y}"
          stroke="black"
          stroke-width="5"></line>

        <!-- y -->
        <line
          x1="0"
          y1="0"
          x2="{cartesianToSVG(0, COORDINATES_SYSTEM_MAX_LENGTH).x}"
          y2="{cartesianToSVG(0, COORDINATES_SYSTEM_MAX_LENGTH).y}"
          stroke="black"
          stroke-width="5"></line>

        <line
          x1="{cartesianToSVG(
            force(q1, q2).F2on1.p1.x,
            force(q1, q2).F2on1.p1.y
          ).x}"
          y1="{cartesianToSVG(
            force(q1, q2).F2on1.p1.x,
            force(q1, q2).F2on1.p1.y
          ).y}"
          x2="{cartesianToSVG(
            force(q1, q2).F2on1.p2.x,
            force(q1, q2).F2on1.p2.y
          ).x}"
          y2="{cartesianToSVG(
            force(q1, q2).F2on1.p2.x,
            force(q1, q2).F2on1.p2.y
          ).y}"
          class="vector {force(q1, q2).F2on1.color}"
          marker-end="url(#arrow-head-blue)"
          stroke-width="5"></line>

        <line
          x1="{cartesianToSVG(
            force(q1, q2).F1on2.p1.x,
            force(q1, q2).F1on2.p1.y
          ).x}"
          y1="{cartesianToSVG(
            force(q1, q2).F1on2.p1.x,
            force(q1, q2).F1on2.p1.y
          ).y}"
          x2="{cartesianToSVG(
            force(q1, q2).F1on2.p2.x,
            force(q1, q2).F1on2.p2.y
          ).x}"
          y2="{cartesianToSVG(
            force(q1, q2).F1on2.p2.x,
            force(q1, q2).F1on2.p2.y
          ).y}"
          class="vector {force(q1, q2).F1on2.color}"
          marker-end="url(#arrow-head-blue)"
          stroke-width="5"></line>

        <line
          x1="{cartesianToSVG(
            force(q1, q3).F2on1.p1.x,
            force(q1, q3).F2on1.p1.y
          ).x}"
          y1="{cartesianToSVG(
            force(q1, q3).F2on1.p1.x,
            force(q1, q3).F2on1.p1.y
          ).y}"
          x2="{cartesianToSVG(
            force(q1, q3).F2on1.p2.x,
            force(q1, q3).F2on1.p2.y
          ).x}"
          y2="{cartesianToSVG(
            force(q1, q3).F2on1.p2.x,
            force(q1, q3).F2on1.p2.y
          ).y}"
          class="vector {force(q1, q2).F1on2.color}"
          marker-end="url(#arrow-head-blue)"
          stroke-width="5"></line>

        <line
          x1="{cartesianToSVG(
            force(q1, q3).F1on2.p1.x,
            force(q1, q3).F1on2.p1.y
          ).x}"
          y1="{cartesianToSVG(
            force(q1, q3).F1on2.p1.x,
            force(q1, q3).F1on2.p1.y
          ).y}"
          x2="{cartesianToSVG(
            force(q1, q3).F1on2.p2.x,
            force(q1, q3).F1on2.p2.y
          ).x}"
          y2="{cartesianToSVG(
            force(q1, q3).F1on2.p2.x,
            force(q1, q3).F1on2.p2.y
          ).y}"
          class="vector {force(q1, q2).F1on2.color}"
          marker-end="url(#arrow-head-blue)"
          stroke-width="5"></line>

        <line
          x1="{cartesianToSVG(
            force(q2, q3).F1on2.p1.x,
            force(q2, q3).F1on2.p1.y
          ).x}"
          y1="{cartesianToSVG(
            force(q2, q3).F1on2.p1.x,
            force(q2, q3).F1on2.p1.y
          ).y}"
          x2="{cartesianToSVG(
            force(q2, q3).F1on2.p2.x,
            force(q2, q3).F1on2.p2.y
          ).x}"
          y2="{cartesianToSVG(
            force(q2, q3).F1on2.p2.x,
            force(q2, q3).F1on2.p2.y
          ).y}"
          class="vector {force(q2, q3).F1on2.color}"
          marker-end="url(#arrow-head-blue)"
          stroke-width="5"></line>

        <line
          x1="{cartesianToSVG(
            force(q2, q3).F2on1.p1.x,
            force(q2, q3).F2on1.p1.y
          ).x}"
          y1="{cartesianToSVG(
            force(q2, q3).F2on1.p1.x,
            force(q2, q3).F2on1.p1.y
          ).y}"
          x2="{cartesianToSVG(
            force(q2, q3).F2on1.p2.x,
            force(q2, q3).F2on1.p2.y
          ).x}"
          y2="{cartesianToSVG(
            force(q2, q3).F2on1.p2.x,
            force(q2, q3).F2on1.p2.y
          ).y}"
          class="vector {force(q2, q3).F2on1.color}"
          marker-end="url(#arrow-head-blue)"
          stroke-width="5"></line>

        <!-- Charges -->
        <circle
          cx="{cartesianToSVG(q1.location.x, q1.location.y).x}"
          cy="{cartesianToSVG(q1.location.x, q1.location.y).y}"
          r="{Math.abs(q1.value)}"
          class="{colorChargeCSS(q1)}"></circle>

        <circle
          cx="{cartesianToSVG(q2.location.x, q2.location.y).x}"
          cy="{cartesianToSVG(q2.location.x, q2.location.y).y}"
          r="{Math.abs(q2.value)}"
          class="{colorChargeCSS(q2)}"></circle>

        <circle
          cx="{cartesianToSVG(q3.location.x, q3.location.y).x}"
          cy="{cartesianToSVG(q3.location.x, q3.location.y).y}"
          r="{Math.abs(q3.value)}"
          class="{colorChargeCSS(q3)}"></circle>

        <!-- End Charges -->

        <!-- <line
        x1="0"
        y1="0"
        x2="50"
        y2="50"
        stroke="#000"
        stroke-width="6"
        marker-end="url(#arrowhead)"></line> -->
      </svg>
    </div>
  </div>
</main>

<style>
main {
  @apply container mx-auto;
}

.charge-blue {
  fill: blue;
  stroke: blue;
  stroke-width: 0.6em;
  /* @apply fill-current text-blue-500; */
}

.charge-red {
  fill: red;
  stroke: red;
  stroke-width: 0.6em;
}

.vector.blue {
  stroke: blue;
}

.vector.red {
  stroke: red;
}
</style>
