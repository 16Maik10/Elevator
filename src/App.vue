<script setup>
import { onBeforeMount, reactive, ref, watch } from "vue";
import ElevatorShaft from "./components/Elevator-shaft.vue";
import ElevatorBtn from "./components/Elevator-btn.vue";
import { countOfFloors, floorHeight, countOfShafts } from "../houseSetting";
const stateOfLifts = ref([]);
const currentFloors = ref([]);
const currentStoppedFloors = ref([]);
const btnsState = reactive({});
const queue = ref([]);
const Funcs = reactive({});
const moveOrQue = (floor) => {
  if (!floor || currentStoppedFloors.value.includes(floor)) {
    return;
  }
  btnsState[floor] = true;
  if (!stateOfLifts.value.includes(true)) {
    !queue.value.includes(floor) && queue.value.push(floor);
  } else {
    const lift = stateOfLifts.value.indexOf(true) + 1;
    currentStoppedFloors.value[lift - 1] = 0;
    Funcs[lift](floor, Math.abs(floor - currentFloors.value[lift - 1]) * 1000);
  }
};
const logState = (data) => {
  const [state, order, currentFloorOfLift, currentStoppedFloorOfLift] = data;
  currentFloors.value[order - 1] = currentFloorOfLift;
  currentStoppedFloors.value[order - 1] = currentStoppedFloorOfLift;
  stateOfLifts.value[order - 1] = state;
  if (stateOfLifts.value.includes(true) && queue.value.length > 0) {
    const lift = stateOfLifts.value.indexOf(true) + 1;
    const floor = queue.value.shift();
    Funcs[lift](floor, Math.abs(floor - currentFloors.value[lift - 1]) * 1000);
  }
};
const setFuncs = (data) => {
  const [func, order] = data;
  Funcs[order] = func;
};
const setBtnState = (floor) => {
  btnsState[floor] = false;
};
watch(stateOfLifts.value, (newStateOfLifts) => {
  sessionStorage.setItem("stateOfLifts", JSON.stringify(newStateOfLifts));
});
watch(btnsState, (newBtnsState) => {
  sessionStorage.setItem("btnsState", JSON.stringify(newBtnsState));
});
watch(queue.value, (newQueue) => {
  sessionStorage.setItem("queue", JSON.stringify(newQueue));
});
watch(currentFloors.value, (newCurrentFloor) => {
  sessionStorage.setItem("currentFloor", JSON.stringify(newCurrentFloor));
});
onBeforeMount(() => {
  const _stateOfLifts = JSON.parse(sessionStorage.getItem("stateOfLifts"));
  for (let i = 1; i <= countOfShafts; i++) {
    currentFloors.value.push(1);
    currentStoppedFloors.value.push(1);
    if (!_stateOfLifts) {
      stateOfLifts.value.push(true);
    }
  }
  if (_stateOfLifts) {
    stateOfLifts.value = _stateOfLifts;
  }
  const _btnsState = JSON.parse(sessionStorage.getItem("btnsState"));
  if (_btnsState) {
    for (let key in _btnsState) {
      btnsState[key] = _btnsState[key];
    }
  } else {
    for (let i = 1; i <= countOfFloors; i++) {
      btnsState[i] = false;
    }
  }
  const _queue = JSON.parse(sessionStorage.getItem("queue"));
  if (_queue) {
    for (let value of _queue) {
      queue.value.push(value);
    }
  }
});
</script>

<template>
  <div class="wrapper">
    <ElevatorShaft
      v-for="n in countOfShafts"
      :key="n"
      :count="countOfFloors"
      :order="n"
      :height="floorHeight"
      @editData="logState"
      @init="setFuncs"
      @setBtnState="setBtnState"
    ></ElevatorShaft>
    <div class="container">
      <ElevatorBtn
        v-for="n in countOfFloors"
        :key="n"
        :number="n"
        :stateOfBtn="btnsState[n]"
        :style="{ height: `${floorHeight}vh` }"
        :action="moveOrQue"
      ></ElevatorBtn>
    </div>
  </div>
</template>

<style lang="scss">
* {
  margin: 0;
  padding: 0;
}

.wrapper {
  display: flex;
}

.container {
  position: relative;
  display: flex;
  flex-direction: column-reverse;
  counter-reset: floor;
  padding-left: 15px;

  &:first-of-type {
    padding-left: 45px;

    & > .elevator {
      left: 46px;
    }
  }
}

.floor {
  width: 100px;
  border-right: 1px solid #000;
  border-left: 1px solid #000;
  position: relative;
  border-bottom: 0.5px solid rgba(128, 128, 128, 0.5);
}

.elevator {
  background-color: #00fefe;
  width: 100px;
  position: absolute;
  bottom: 0.5vh;
  left: 15px;
  transition: bottom 1s linear;

  &_rest {
    animation: waiting 3s linear;
  }

  & > p {
    display: none;
    justify-content: space-between;
    width: 25%;
    margin: 10px auto;
    background-color: rgba(70, 149, 149, 0.5);
    color: #fff;
    padding: 5px 10px;
    border-radius: 10%;
  }

  & > .showDirection {
    display: flex;
  }
}

@keyframes waiting {
  from {
    opacity: 0.25;
  }

  20% {
    opacity: 1;
  }

  40% {
    opacity: 0.25;
  }

  60% {
    opacity: 1;
  }

  80% {
    opacity: 0.25;
  }

  to {
    opacity: 1;
  }
}
</style>
