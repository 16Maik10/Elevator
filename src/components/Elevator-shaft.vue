<script setup>/* eslint-disable */
import { watch, computed, onBeforeMount, onMounted, onUpdated, ref } from "vue";

const props = defineProps(
    ['count',
    'order',
    'height',
    ])
const position = ref(0.5);
const currentFloor = ref(1);
const directionFloor = ref(0);
const liftDirection = ref(true);
const currentStopedFloor = ref(1);
const lifting = ref(false);
const elevatorIsRest = ref(false);
const emit = defineEmits(["editData", "init", "setBtnState"])
const isLiftFree = computed(() => {
    return (!elevatorIsRest.value && !lifting.value);
})
const moving = () => {
  if (liftDirection.value) {
    position.value += props.height;
    currentFloor.value++;
  } else {
    position.value -= props.height;
    currentFloor.value--;
  }
};
const setPosition = (floor, currentTiming, wasMoving = false) => {
  currentStopedFloor.value = 0;
  lifting.value = true;
  directionFloor.value = floor;
  liftDirection.value = floor > currentFloor.value;
  let IID;
  if (!wasMoving) {
    moving();
    IID = setInterval(moving, 1001);
  } else {
    IID = setInterval(moving, 999);
  }
  setTimeout(() => {
    clearInterval(IID);
    emit('setBtnState', currentFloor.value);
    currentStopedFloor.value = currentFloor.value;
    setTimeout(() => {
      elevatorIsRest.value = false;
      emit('editData', [isLiftFree.value, props.order, currentFloor.value, currentStopedFloor.value]);
    }, 3000);
    lifting.value = false;
    elevatorIsRest.value = true;
  }, currentTiming);
};
watch(directionFloor, (newDirectionFloor) => {
  sessionStorage.setItem(`directionFloor${props.order}`, newDirectionFloor);
});
watch(lifting, (newLifting) => {
  sessionStorage.setItem(`lifting${props.order}`, newLifting);
});
onBeforeMount(() => {
    emit('init', [setPosition, props.order]);
    const _currentFloors = JSON.parse(sessionStorage.getItem("currentFloor"));
    if (_currentFloors) {
      currentFloor.value = _currentFloors[props.order - 1];
      position.value = 0.5 + (currentFloor.value - 1) * props.height;
    }
  } 
)
onMounted(() => {
  const _directionFloor = +sessionStorage.getItem(`directionFloor${props.order}`);
  if (_directionFloor) {
    directionFloor.value = _directionFloor;
    liftDirection.value = directionFloor.value > currentFloor.value;
    const _lifting = JSON.parse(sessionStorage.getItem(`lifting${props.order}`));
    lifting.value = _lifting;
    if (lifting.value && Math.abs(directionFloor.value - currentFloor.value) > 1 ) {
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
onUpdated(() => {
  emit('editData', [isLiftFree.value, props.order, currentFloor.value, currentStopedFloor.value]);
})
</script>
<template>
    <div class="container">
        <div class="floor" v-for="n in count" :key="n" :style="{ height: `${height}vh` }">
        </div>
        <div class="elevator" :class="{ elevator_rest: elevatorIsRest }" :style="{
            bottom: `${position}vh`,
            height: `${height}vh`,
        }">
            <p :class="{ showDirection: lifting || elevatorIsRest }">
                <span v-if="liftDirection">&uarr;</span>
                <span v-else>&darr;</span>
                <span>{{ directionFloor }}</span>
            </p>
        </div>
    </div>
</template>