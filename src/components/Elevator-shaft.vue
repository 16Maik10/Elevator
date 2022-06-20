<script setup>/* eslint-disable */
const props = defineProps(
    ['count',
    'height',
    'elevatorIsRest',
    'position',
    'lifting',
    'directionFloor',
    'liftDirection'
    ])
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