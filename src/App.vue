<script setup>
import { onBeforeMount, onMounted, reactive, ref, watch } from "vue";
const position = ref(0.5);
const currentFloor = ref(1);
const directionFloor = ref(0);
const liftDirection = ref(true);
const currentStopedFloor = ref(1);
const btnsState = reactive({
  1: false,
  2: false,
  3: false,
  4: false,
  5: false,
});
const lifting = ref(false);
const elevatorIsRest = ref(false);
const queue = ref([]);
const moving = () => {
  if (liftDirection.value) {
    position.value += 20;
    currentFloor.value++;
  } else {
    position.value -= 20;
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
    position.value = 0.5 + (currentFloor.value - 1) * 20;
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
  <div class="container">
    <div class="floor">
      <button :class="{ btn_active: btnsState[1] }" @click="moveOrQue(1)">
        <div></div>
      </button>
    </div>
    <div class="floor">
      <button :class="{ btn_active: btnsState[2] }" @click="moveOrQue(2)">
        <div></div>
      </button>
    </div>
    <div class="floor">
      <button :class="{ btn_active: btnsState[3] }" @click="moveOrQue(3)">
        <div></div>
      </button>
    </div>
    <div class="floor">
      <button :class="{ btn_active: btnsState[4] }" @click="moveOrQue(4)">
        <div></div>
      </button>
    </div>
    <div class="floor">
      <button :class="{ btn_active: btnsState[5] }" @click="moveOrQue(5)">
        <div></div>
      </button>
    </div>
    <div
      class="elevator"
      :class="{ elevator_rest: elevatorIsRest }"
      :style="{
        bottom: `${position}vh`,
      }"
    >
      <p :class="{ showDirection: lifting || elevatorIsRest }">
        <span v-if="liftDirection">&uarr;</span>
        <span v-else>&darr;</span>
        <span>{{ directionFloor }}</span>
      </p>
    </div>
  </div>
</template>

<style lang="scss">
* {
  margin: 0;
  padding: 0;
}

.container {
  position: relative;
  display: flex;
  flex-direction: column-reverse;
  padding-left: 45px;
  counter-reset: floor;
}

.floor {
  width: 100px;
  height: 20vh;
  border-right: 1px solid #000;
  border-left: 1px solid #000;
  position: relative;
  border-bottom: 0.5px solid rgba(128, 128, 128, 0.5);

  & > button {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 25px;
    height: 25px;
    background-color: #fff;
    position: absolute;
    counter-increment: floor;
    top: 10vh;
    right: -50px;
    cursor: pointer;
    font-size: 18px;
    border: 1px solid #0079a7;
    border-radius: 4px;
    transition: box-shadow 0.2s linear, border-color 0.2s linear;

    &:hover {
      box-shadow: 0px 0px 5px #0079a7;
    }

    & > div {
      width: 50%;
      height: 50%;
      border-radius: 100%;
      background-color: #0079a7;
      transition: background-color 0.3s;
    }

    &::after {
      content: counter(floor);
      position: absolute;
      top: -2rem;
      left: 0;
    }
  }

  & > .btn_active {
    border-color: #e25300;

    &:hover {
      box-shadow: 0px 0px 5px #e25300;
    }

    & > div {
      background-color: #e25300;
    }
  }
}

.elevator {
  background-color: #00fefe;
  width: 100px;
  height: 20vh;
  position: absolute;
  bottom: 0.5vh;
  left: 46px;
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
