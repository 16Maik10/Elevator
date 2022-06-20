<script setup>/* eslint-disable */
const props = defineProps(['action','number','stateOfBtn'])
</script>
<template>
    <div class="btn-wrapper">
        <button @click="action(number)" :class="{ btn_active: stateOfBtn }">
        <div></div>
      </button>
    </div>
</template>

<style lang="scss" scoped>
.btn-wrapper{
    display: flex;
    align-items: center;
    justify-content: center;
    width: 5vw;
}

button {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 25px;
    height: 25px;
    background-color: #fff;
    counter-increment: floor;
    position: relative;
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
      top: -1.5rem;
      left: 0;
    }
  }

    .btn_active {
    border-color: #e25300;

    &:hover {
      box-shadow: 0px 0px 5px #e25300;
    }

    & > div {
      background-color: #e25300;
    }
  }
</style>