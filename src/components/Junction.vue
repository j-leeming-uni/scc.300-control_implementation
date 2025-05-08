<script setup lang="ts">
import {computed, ref} from 'vue'
import {useIntervalFn} from '@vueuse/core'

const north = ref(false)
const south = ref(false)
const east = ref(false)
const west = ref(false)

const northCount = ref(0)
const southCount = ref(0)
const eastCount = ref(0)
const westCount = ref(0)

const sinceNorthReleased = ref(0)
const sinceEastReleased = ref(0)
const sinceSouthReleased = ref(0)
const sinceWestReleased = ref(0)

const emit = defineEmits<{
    (e: 'log', message: string): void
    (e: 'hint', direction: string): void
}>()
const props = defineProps<{
    jid: number
    attacks: {
        dos: boolean
        disclosure: boolean
        spoofing: boolean
        repudiation: boolean
    }
    hints: {
        north: boolean
        east: boolean
        south: boolean
        west: boolean
    }
}>()

const currentDirection = ref(0)

const northStyle = computed(() => `--color: ${north.value ? 'green' : 'red'}; --weight: ${props.hints.north ? 'bold' : 'normal'}`)
const southStyle = computed(() => `--color: ${south.value ? 'green' : 'red'}; --weight: ${props.hints.south ? 'bold' : 'normal'}`)
const eastStyle = computed(() => `--color: ${east.value ? 'green' : 'red'}; --weight: ${props.hints.east ? 'bold' : 'normal'}`)
const westStyle = computed(() => `--color: ${west.value ? 'green' : 'red'}; --weight: ${props.hints.west ? 'bold' : 'normal'}`)

const VISIT_THRESHOLD = 10
useIntervalFn(() => {
    if (props.attacks.dos) {
        north.value = false
        east.value = false
        south.value = false
        west.value = false
        return
    }

    const queues = [northCount, eastCount, southCount, westCount]
    const maxQueue = Math.max(...queues.map((queue) => queue.value))
    const indices = queues.map((queue, index) => queue.value === maxQueue ? index : -1).filter((index) => index !== -1)

    if (props.hints.north) sinceNorthReleased.value = VISIT_THRESHOLD - 2
    if (props.hints.east) sinceEastReleased.value = VISIT_THRESHOLD - 2
    if (props.hints.south) sinceSouthReleased.value = VISIT_THRESHOLD - 2
    if (props.hints.west) sinceWestReleased.value = VISIT_THRESHOLD - 2

    currentDirection.value = (currentDirection.value + 1) % (indices.length || 4)

    sinceNorthReleased.value++
    sinceEastReleased.value++
    sinceSouthReleased.value++
    sinceWestReleased.value++

    let smartRelease = true
    if (sinceNorthReleased.value > VISIT_THRESHOLD) {
        currentDirection.value = 0
        smartRelease = false
    } else if (sinceEastReleased.value > VISIT_THRESHOLD) {
        currentDirection.value = 1
        smartRelease = false
    } else if (sinceSouthReleased.value > VISIT_THRESHOLD) {
        currentDirection.value = 2
        smartRelease = false
    } else if (sinceWestReleased.value > VISIT_THRESHOLD) {
        currentDirection.value = 3
        smartRelease = false
    }

    const direction = smartRelease ? indices[currentDirection.value] : currentDirection.value
    north.value = direction === 0
    east.value = direction === 1
    south.value = direction === 2
    west.value = direction === 3

    if (north.value) {
        emit('hint', 'north')
        sinceNorthReleased.value = 0
    }
    if (east.value) {
        emit('hint', 'east')
        sinceEastReleased.value = 0
    }
    if (south.value) {
        emit('hint', 'south')
        sinceSouthReleased.value = 0
    }
    if (west.value) {
        emit('hint', 'west')
        sinceWestReleased.value = 0
    }

    const logs = [
        `Junction ${props.jid}`,
        `  North:`,
        `    ${northCount.value} vehicles`,
        `    ${north.value ? 'Green' : 'Red'}`,
        `    ${sinceNorthReleased.value} seconds since last release`,
        `  East:`,
        `    ${eastCount.value} vehicles`,
        `    ${east.value ? 'Green' : 'Red'}`,
        `    ${sinceEastReleased.value} seconds since last release`,
        `  South:`,
        `    ${southCount.value} vehicles`,
        `    ${south.value ? 'Green' : 'Red'}`,
        `    ${sinceSouthReleased.value} seconds since last release`,
        `  West:`,
        `    ${westCount.value} vehicles`,
        `    ${west.value ? 'Green' : 'Red'}`,
        `    ${sinceWestReleased.value} seconds since last release`
    ].join('\n')
    if (props.attacks.disclosure) {
        emit('log', logs)
    }
}, 1000)
</script>

<template>
    <div class="junction">
        <div class="id">J{{ jid }}</div>

        <div class="light north" :style="northStyle">N</div>
        <div class="light south" :style="southStyle">S</div>
        <div class="light east" :style="eastStyle">E</div>
        <div class="light west" :style="westStyle">W</div>

        <input type="number" v-model="northCount" class="queue north"/>
        <input type="number" v-model="southCount" class="queue south"/>
        <input type="number" v-model="eastCount" class="queue east"/>
        <input type="number" v-model="westCount" class="queue west"/>
    </div>
</template>

<style scoped>
.junction {
    position: relative;
    width: 12rem;
    height: 12rem;
}

.junction .id {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-size: 2rem;
}

.light {
    position: absolute;
    width: 4rem;
    height: 4rem;
    border-radius: 50%;
    background-color: var(--color);
    font-weight: var(--weight);

    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 2rem;
}

.light.north {
    top: 0;
    left: 50%;
    transform: translateX(-50%);
}

.light.south {
    bottom: 0;
    left: 50%;
    transform: translateX(-50%);
}

.light.east {
    top: 50%;
    right: 0;
    transform: translateY(-50%);
}

.light.west {
    top: 50%;
    left: 0;
    transform: translateY(-50%);
}

.queue {
    position: absolute;
    width: 4rem;
    height: 3rem;
    border-radius: 10%;
    color: black;
    background-color: white;
    font-size: 1rem;
    text-align: center;
}

.queue.north {
    top: 0;
    left: 50%;
    transform: translate(-50%, -125%);
}

.queue.south {
    bottom: 0;
    left: 50%;
    transform: translate(-50%, 125%);
}

.queue.east {
    top: 50%;
    right: 0;
    transform: translate(125%, -50%);
}

.queue.west {
    top: 50%;
    left: 0;
    transform: translate(-125%, -50%);
}
</style>