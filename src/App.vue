<script setup lang="ts">
import {computed, ref, watch} from 'vue'
import Junction from './components/Junction.vue'

const dos = ref(false)
const disclosure = ref(false)
const spoofing = ref(false)
const repudiation = ref(false)

const attacks = computed(() => ({
    dos: dos.value,
    disclosure: disclosure.value,
    spoofing: spoofing.value,
    repudiation: repudiation.value
}))

const logs = ref([])

watch(disclosure, () => {
    if (!disclosure.value) {
        logs.value = []
    }
})

const nullHint = () => ({
    north: null,
    east: null,
    south: null,
    west: null
})

const hints = ref({
    1: nullHint(),
    2: nullHint(),
})

function onHint(from, to, direction) {
    if (to === 2) {
        hints.value[2] = nullHint()
        if (direction) hints.value[2][direction] = true
    } else if (to === 1) {
        hints.value[1] = nullHint()
        if (direction) hints.value[1][direction] = true
    }
}

const spoofTarget = ref(1)
const spoofAs = ref(2)
const spoofDirection = ref('north')
</script>

<template>
    <div class="row">
        <Junction :jid="1" :attacks="attacks" :hints="hints[1]" @log="value => logs.push(value)"
                  @hint="direction => onHint(1, 2, direction === 'west' ? direction : null)"/>
        <Junction :jid="2" :attacks="attacks" :hints="hints[2]" @log="value => logs.push(value)"
                  @hint="direction => onHint(2, 1, direction === 'east' ? direction : null)"/>

        <div class="logs" v-if="logs.length">
            <template v-for="log in logs">
                <pre>{{ log }}</pre>
            </template>
        </div>
    </div>

    <div class="controls">
        <div>
            <input id="dos" type="checkbox" v-model="dos"/>
            <label for="dos">DoS</label>
        </div>

        <div>
            <input id="disclosure" type="checkbox" v-model="disclosure"/>
            <label for="disclosure">Information Disclosure</label>
        </div>

        <div class="spoofing">
            <p>Spoofing / Repudiation</p>
            <label for="spoofing-target">Target</label>
            <input id="spoofing-target" type="number" v-model="spoofTarget"/>
            <label for="spoofing-as">as</label>
            <input id="spoofing-as" type="number" v-model="spoofAs"/>

            <select v-model="spoofDirection">
                <option value="north">North</option>
                <option value="east">East</option>
                <option value="south">South</option>
                <option value="west">West</option>
            </select>

            <button @click="onHint(spoofAs, spoofTarget, spoofDirection)">Spoof</button>
        </div>
    </div>
</template>

<style scoped>
.row {
    height: 60vh;
    display: flex;
    align-items: center;
    justify-content: space-evenly;
    gap: 16rem;
    margin-block: 4rem;
}

.controls {
    display: flex;
    justify-content: center;
    align-items: start;
    gap: 2rem;
    margin-bottom: 4rem;
}

.controls * {
    padding: 0;
    margin: 0;
}

.controls label {
    margin-inline: 0.5rem;
}

.spoofing {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
}

.logs {
    padding-right: 2rem;
    max-height: 100%;
    overflow-y: scroll;
    text-align: left;
}
</style>
