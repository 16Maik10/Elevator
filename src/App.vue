<script setup>
import { onBeforeMount, onMounted, reactive, ref, watch } from "vue";
import ElevatorShaft from "./components/Elevator-shaft.vue";
import ElevatorBtn from "./components/Elevator-btn.vue";
import { countOfFloors, floorHeight } from "../houseSetting";
const position = ref(0.5);
const currentFloor = ref(1);
const directionFloor = ref(0);
const liftDirection = ref(true);
const currentStopedFloor = ref(1);
const btnsState = reactive({});
const lifting = ref(false);
const elevatorIsRest = ref(false);
const queue = ref([]);
const moving = () => {
  if (liftDirection.value) {
    position.value += floorHeight;
    currentFloor.value++;
  } else {
    position.value -= floorHeight;
    currentFloor.value--;
  }
};
const setPosition = (floor, currentTiming, wasMoving = false) => {
  lifting.value = true;
  directionFloor.value = floor;
  if (!wasMoving) {
    moving();
  }
  const IID = setInterval(moving, 1001);
  setTimeout(() => {
    clearInterval(IID);
    btnsState[floor] = false;
    currentStopedFloor.value = currentFloor.value;
    setTimeout(() => {
      elevatorIsRest.value = false;
      moveOrQue(queue.value.shift());
    }, 3000);
    lifting.value = false;
    elevatorIsRest.value = true;
  }, currentTiming);
};
const moveOrQue = (floor) => {
  if (!floor || floor === currentStopedFloor.value) {
    return;
  }
  btnsState[floor] = true;
  currentStopedFloor.value = 0;
  if (lifting.value || elevatorIsRest.value) {
    !queue.value.includes(floor) && queue.value.push(floor);
  } else {
    liftDirection.value = floor > currentFloor.value;
    setPosition(floor, Math.abs(floor - currentFloor.value) * 1000);
  }
};
watch(btnsState, (newBtnsState) => {
  sessionStorage.setItem("btnsState", JSON.stringify(newBtnsState));
});
watch(queue.value, (newQueue) => {
  sessionStorage.setItem("queue", JSON.stringify(newQueue));
});
watch(directionFloor, (newDirectionFloor) => {
  sessionStorage.setItem("directionFloor", newDirectionFloor);
});
watch(currentFloor, (newCurrentFloor) => {
  sessionStorage.setItem("currentFloor", newCurrentFloor);
});
watch(lifting, (newLifting) => {
  sessionStorage.setItem("lifting", newLifting);
});
onBeforeMount(() => {
  for (let i = 1; i <= countOfFloors; i++) {
    btnsState[i] = false;
  }
  const _btnsState = JSON.parse(sessionStorage.getItem("btnsState"));
  if (_btnsState) {
    for (let key in _btnsState) {
      btnsState[key] = _btnsState[key];
    }
  }
  const _queue = JSON.parse(sessionStorage.getItem("queue"));
  if (_queue) {
    for (let value of _queue) {
      queue.value.push(value);
    }
  }
  const _currentFloor = +sessionStorage.getItem("currentFloor");
  if (_currentFloor) {
    currentFloor.value = _currentFloor;
    position.value = 0.5 + (currentFloor.value - 1) * floorHeight;
  }
});
onMounted(() => {
  const _directionFloor = +sessionStorage.getItem("directionFloor");
  if (_directionFloor) {
    liftDirection.value = _directionFloor > currentFloor.value;
    const _lifting = sessionStorage.getItem("lifting");
    if (_lifting === "true") {
      setPosition(
        _directionFloor,
        Math.abs(_directionFloor - currentFloor.value) * 1000
      );
    } else {
      setPosition(
        _directionFloor,
        Math.abs(_directionFloor - currentFloor.value) * 1000,
        true
      );
    }
  }
});
</script>

<template>
  <div class="wrapper">
    <ElevatorShaft
      v-for="n in 4"
      :key="n"
      :count="countOfFloors"
      :height="floorHeight"
      :elevatorIsRest="elevatorIsRest"
      :position="position"
      :lifting="lifting"
      :directionFloor="directionFloor"
      :liftDirection="liftDirection"
    ></ElevatorShaft>
    <!-- <div class="floor">
      <button :class="{ btn_active: btnsState[4] }" @click="moveOrQue(4)">
        <div></div>
      </button>
    </div> -->
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
