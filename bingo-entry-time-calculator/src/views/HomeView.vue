<script setup>
import worldsData from '../data/shineSpeeds.json'
import shineImage from '@/assets/shine.png'
import coolShineImage from '@/assets/shine-cool.png'
import pollutedImage from '@/assets/polluted.png'
import blueCoinImage from '@/assets/blue-coin.png'
import shadowMarioImage from '@/assets/shadow-mario.png'
import canonImage from '@/assets/pinna-cannon.png'
import pipeImage from '@/assets/pipe.png'
import pollutedImage2 from '@/assets/polluted2.png'

import SelectInput from '@/components/SelectInput.vue'
import { computed, ref } from 'vue'

const worlds = ref(structuredClone(worldsData))
const eventCutsceneAmount = ref()
const transitions = ref()
const misc = ref()
const totalShines = ref()
const isCompact = ref(0)
const showSummary = ref(true)
const whenevers = ref()
const shineSize = ref('medium')
const timeLog = ref([])

function toggleSelected(world, shine) {
  const target = worlds.value[world].data[shine]
  const wasSelected = target.selected === 1 || target.selected === 2
  if (target.selected === 0) {
    worlds.value[world].requiredUnlocks?.forEach((unlockName) => {
      const unlockWorld = worlds.value.Unlocks?.data
      if (unlockWorld) {
        const unlockEntry = Object.values(unlockWorld).find((u) => u.name === unlockName)
        if (unlockEntry && unlockEntry.selected === 0) {
          unlockEntry.selected = 1
          addLogEntry(unlockEntry.name, true)
        }
      }
    })
  }

  if (target.sequential && !wasSelected) {
    const shines = Array.from({ length: shine }, (_, i) => i + 1)
    shines.forEach((shine) => {
      const secondTarget = worlds.value[world].data[shine]
      if (secondTarget.sequential && !secondTarget.selected) {
        if (target.name === 'Gelato Unlock' && secondTarget.name === 'Ricco Unlock') return
        secondTarget.selected = 1
        addLogEntry(secondTarget.name, worlds.value[world].type === 'unlocks')
      }
    })
  } else if (target.selected === 1 && target.secondaryName) {
    target.selected = 2
    const index = timeLog.value.findIndex((e) => Object.keys(e)[0] === target.name)
    if (index != -1) {
      const oldTime = Object.values(timeLog.value[index])[0]
      const diff = mmssToSeconds(target.secondaryTime) - mmssToSeconds(target.time)
      timeLog.value[index] = { [target.secondaryName]: oldTime + diff }
    }
  } else if (target.selected === 0) {
    target.selected = 1
    addLogEntry(target.name, worlds.value[world].type === 'unlocks')
  } else {
    if (target.selected === 1) {
      deleteLogEntry(target.name)
    } else {
      deleteLogEntry(target.secondaryName)
    }
    target.selected = 0
  }
  timeLog.value = timeLog.value.filter((entry) => {
    const key = Object.keys(entry)[0]
    const isShineCountLog = /^\d+ shines$/.test(key)
    if (!isShineCountLog) return true
    const count = Number(key.split(' ')[0])
    return count <= shineCount.value
  })
}

function mmssToSeconds(timeString) {
  const [m, s] = timeString.split(':').map(Number)
  return (m || 0) * 60 + (s || 0)
}

function secondsToMMSS(seconds) {
  if (!seconds || seconds <= 0) return '0:00'
  const m = Math.floor(seconds / 60)
  const s = Math.floor(seconds % 60)
  return `${m}:${s.toString().padStart(2, '0')}`
}

const countTotals = () => {
  const transitionSeconds = Number(transitions.value ?? 0)
  const tradeSeconds = totalShines.value > 0 ? 16 : 0
  const miscSeconds = Number(misc.value ?? 0) + tradeSeconds
  const wheneverSeconds = Number(whenevers.value ?? 0) * 10
  const eventCutsceneSeconds = Number(eventCutsceneAmount.value ?? 0) * 4
  const total =
    transitionSeconds +
    miscSeconds +
    wheneverSeconds +
    eventCutsceneSeconds -
    (shineCount.value >= 2 ? 12 : 0) //fix shine get -> plaza spawn for 1st shine
  const worldTotals = {
    ...(worlds.value.Delfino.data[1].selected
      ? { Cutscenes: mmssToSeconds(worlds.value.Delfino.data[1].time) }
      : {}),
    ...(eventCutsceneSeconds > 0 ? { 'Event CS': eventCutsceneSeconds } : {}),
    ...(transitionSeconds > 0 ? { Transitions: transitionSeconds } : {}),
    ...(miscSeconds > 0 ? { Misc: miscSeconds } : {}),
    ...(wheneverSeconds > 0 ? { Whenevers: wheneverSeconds } : {}),
  }
  return { total, worldTotals }
}

const totals = computed(() => {
  let { total, worldTotals } = countTotals()

  Object.entries(worlds.value).forEach(([worldName, world]) => {
    if (world?.data) {
      let worldTotal = 0
      Object.values(world.data).forEach((shine) => {
        const time = shine.selected === 2 ? shine.secondaryTime : shine.time
        if (shine.selected && shine.time) {
          total += mmssToSeconds(time)
          if (worldName === 'Delfino' && shine.name === 'Airstrip') return
          worldTotal += mmssToSeconds(time)
        }
      })
      if (worldTotal > 0) {
        worldTotals[worldName] = worldTotal
      }
    }
  })
  return {
    total,
    worldTotals,
  }
})

const shineCount = computed(() => {
  let count = totalShines.value ?? 0
  Object.values(worlds.value).forEach((world) => {
    if (world?.data && world.type === 'shines') {
      Object.values(world.data).forEach((shine) => {
        if (shine.selected) {
          count++
        }
      })
    }
  })

  return count
})

function loadCustomJson(event) {
  const file = event.target.files[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = (e) => {
    try {
      const parsed = JSON.parse(e.target.result)
      if (parsed && typeof parsed === 'object' && Object.keys(parsed).length) {
        worlds.value = parsed
      }
    } catch (err) {}
  }
  reader.readAsText(file)
}

function toggleWorld(worldName) {
  worlds.value[worldName].isOpen = !worlds.value[worldName].isOpen
}

function alterCompact(currentValue) {
  if (currentValue === 0 || currentValue === 1) {
    isCompact.value += 1
  } else {
    isCompact.value = 0
  }
}

function isWorldEmpty(world) {
  if (!world?.data) return true
  return !Object.values(world.data).some((shine) => shine.selected)
}

function addLogEntry(shineName, isUnlock) {
  let { total } = countTotals()
  Object.entries(worlds.value).forEach(([worldName, world]) => {
    if (world?.data) {
      Object.values(world.data).forEach((shine) => {
        const time = shine.selected === 2 ? shine.secondaryTime : shine.time
        if (shine.selected && shine.time) {
          total += mmssToSeconds(time)
        }
      })
    }
  })
  timeLog.value.push({ [shineName]: total })
  const thresholds = [3, 5, 10, 20, 26]
  if (thresholds.includes(shineCount.value) && !isUnlock) {
    console.log(shineCount.value)
    timeLog.value.push({ [shineCount.value + ' shines']: total })
  }
}

function deleteLogEntry(shineName) {
  const index = timeLog.value.findIndex((entry) => Object.keys(entry)[0] === shineName)
  if (index !== -1) {
    timeLog.value.splice(index, 1)
  }
}
</script>

<template>
  <div class="layout">
    <div class="container">
      <label class="shine-size-label">
        Shine size:
        <select v-model="shineSize" class="shine-size-select">
          <option value="small">Small</option>
          <option value="medium">Medium</option>
          <option value="large">Large</option>
        </select>
      </label>
      <h1 class="value">Route Time Calculator</h1>

      <label class="upload-btn">
        Upload custom shine speeds
        <input type="file" accept="application/json" @change="loadCustomJson" />
      </label>
      <button class="compact-btn" :class="{ active: isCompact }" @click="alterCompact(isCompact)">
        {{
          isCompact === 0 ? 'Compact View' : isCompact === 1 ? 'Screenshot View' : 'Default View'
        }}
      </button>
      <router-link to="/instructions" class="compact-btn"> App Instructions </router-link>
      <div v-for="(shines, worldName) in worlds" :key="worldName" class="{world-row: isCompact }">
        <h2 class="world-title clickable" v-if="isCompact === 0" @click="toggleWorld(worldName)">
          {{ worldName }}
        </h2>
        <div
          :class="['shine-grid', { 'longer-gap': worldName === 'Unlocks' }]"
          v-if="shines.isOpen && (isCompact !== 2 || !isWorldEmpty(shines))"
        >
          <div v-for="(shine, index) in shines.data" :key="shine" class="shine-item">
            <img
              :src="
                shines.type == 'shines'
                  ? shine.selected === 2
                    ? coolShineImage
                    : shineImage
                  : shines.type === 'blues'
                    ? blueCoinImage
                    : shine.name === 'Shadow Mario Secret Cutscene'
                      ? shadowMarioImage
                      : shine.name === 'Pinna Unlock'
                        ? canonImage
                        : shine.name === 'Sirena Unlock'
                          ? pipeImage
                          : shine.name === 'Ricco Unlock' || shine.name === 'Gelato Unlock'
                            ? pollutedImage2
                            : pollutedImage
              "
              :alt="shine.name"
              :class="[
                'shine-img',
                shineSize,
                {
                  selected: shine.selected === 1 || shine.selected === 2,
                  notSelected: shine.selected === 0,
                },
                { larger: worldName === 'Unlocks' },
              ]"
              @click="toggleSelected(worldName, index)"
            />
            <span class="shine-name">{{
              shine.selected === 2 ? shine.secondaryName : shine.name
            }}</span>
          </div>
        </div>
      </div>
      <h2 class="world-title">Other</h2>

      <div class="input-row">
        <SelectInput
          label="Event Cutscenes"
          :options="[
            { value: '3', label: 'Gelato unlock min' },
            { value: '4', label: 'Pinna unlock min' },
            { value: '6', label: 'Noki unlock, unlock yoshi asap' },
            { value: '11', label: 'Noki unlock, yoshi at 14 no unlock' },
          ]"
          v-model="eventCutsceneAmount"
        />
        <SelectInput label="Sirena Whenevers" v-model="whenevers" />
        <SelectInput
          label="Transitions(seconds)"
          :options="[
            { value: '5', label: 'Bianco -> Ricco (or reverse)' },
            { value: '9', label: 'Bianco -> Gelato (or reverse)' },
            { value: '13', label: 'Gelato -> Ricco (or reverse)' },
            { value: '15', label: 'Default Start -> Pianta' },
            { value: '9', label: 'Pinna -> Ricco' },
          ]"
          v-model="transitions"
        />

        <SelectInput label="Misc timeloss(seconds)" v-model="misc" />
        <SelectInput label="Shines from Blue Trades" v-model="totalShines" />
      </div>
    </div>
    <div class="columnContainer">
      <div class="totals">
        <button
          class="compact-btn"
          :class="{ active: showSummary }"
          @click="showSummary = !showSummary"
        >
          {{ showSummary ? 'Hide' : 'Show' }} world summaries
        </button>
        <div v-if="showSummary">
          <p v-for="(time, world) in totals.worldTotals" :key="world" class="total-line">
            <span class="label-text">{{ world }}:</span>
            <span class="value">{{ secondsToMMSS(time) }}</span>
          </p>
          <hr style="width: 80%" />
        </div>
        <p class="total-line">
          <span class="label-text">Total Time:</span>
          <span class="value">{{ secondsToMMSS(totals.total) }}</span>
        </p>
        <p class="total-line">
          <span class="label-text">Total Shines:</span>
          <span class="value">{{ shineCount }}</span>
        </p>
      </div>

      <div class="shine-log">
        <h3 class="shine-log-title">Progress Log</h3>
        <div class="shine-log-messages">
          <div v-for="(entry, i) in timeLog" :key="i" class="shine-log-entry">
            {{ Object.keys(entry)[0] }}
            {{ Object.keys(entry)[0].includes('Unlock') ? 'done' : 'collected' }} at
            {{ secondsToMMSS(Object.values(entry)[0]) }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.layout {
  display: flex;
  width: 100%;
  max-width: 1500px;
  box-sizing: border-box;
  justify-content: space-between;
  align-items: flex-start;
  gap: 2rem;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}
.container {
  flex: 1;
  max-width: 900px;
}

.columnContainer {
  display: flex;
  flex-direction: column;
  flex: 0 0 350px;
  min-width: 280px;
  max-width: 400px;
}

@media (max-width: 1000px) {
  .layout {
    flex-direction: column;
    align-items: center;
  }

  .container,
  .columnContainer {
    width: 100%;
    max-width: 1000px;
  }
}

h1 {
  text-align: center;
  font-size: 26px;
  margin-bottom: 20px;
}

hr {
  margin-bottom: 10px;
}

.world-row {
  margin-bottom: 15px;
}

.world-row h2 {
  margin-bottom: 10px;
  font-size: 20px;
  border-bottom: 2px solid #ffffff;
  margin-bottom: 0.75rem;
  padding-bottom: 4px;
}

.shine-grid {
  display: grid;
  grid-template-columns: repeat(11, 1fr);
  gap: 10px;
}

.shine-grid.longer-gap {
  gap: 45px;
}

.shine-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.shine-img {
  width: 48px;
  height: 48px;
  cursor: pointer;
  transition: opacity 0.2s ease;
}

.shine-img.small {
  width: 24px;
  height: 24px;
}

.shine-img.large {
  width: 64px;
  height: 64px;
}

.shine-img.larger {
  width: 96px;
  height: 96px;
}

.selected {
  filter: brightness(1.1) drop-shadow(0 0 8px rgba(255, 220, 0, 0.8));
  transform: scale(1.05);
  transition: all 0.15s ease;
}

.notSelected {
  filter: grayscale(100%);
  background-color: transparent;
}

.shine-img:hover {
  opacity: 0.8;
}

.shine-name {
  font-family: 'Poppins', sans-serif;
  font-weight: 500;
  font-size: 0.9rem;
  color: #3a3a3a;
  text-shadow: 0 0 3px rgba(255, 255, 255, 0.7);
  font-weight: 500;
  margin-top: 0.3rem;
  white-space: nowrap;
  white-space: normal;
  max-width: 90px;
  line-height: 1.1;
}

.input-row {
  display: flex;
  gap: 1rem;
  align-items: flex-start;
  margin-bottom: 30px;
}

.totals {
  display: inline-block;
  text-align: left;
  margin-top: 5.2rem;
  margin-left: 3rem;
}

.total-line {
  margin: 0.4rem 0;
  display: flex;
  justify-content: flex-start;
  align-items: baseline;
  white-space: nowrap;
  gap: 10px;
}

.label-text {
  color: #ffffff;
  font-weight: 600;
  font-size: 1.8rem;
  text-shadow: 0 0 6px rgba(0, 0, 0, 0.4);
  display: inline-block;
  width: 160px;
  text-align: right;
  margin-right: 10px;
}

.value {
  color: #ffd700;
  font-weight: 700;
  font-size: 2.4rem;
  text-shadow:
    -1px -1px 0 #444,
    1px -1px 0 #444,
    -1px 1px 0 #444,
    1px 1px 0 #444;
  margin-left: 0.25rem;
}

.world-title {
  font-family: 'Poppins', sans-serif;
  font-weight: 700;
  font-size: 1.8rem;
  color: #ffffff;
  text-shadow:
    2px 2px 0 #0078ff,
    4px 4px 6px rgba(0, 0, 0, 0.4);
  margin-bottom: 0.3rem;
  border-bottom: 2px solid rgba(255, 220, 150, 0.8);
  padding-bottom: 0.3rem;
  cursor: pointer;
  user-select: none;
}

.world-title:hover {
  color: #ffcc00;
}

.upload-btn {
  display: inline-block;
  background: linear-gradient(180deg, #ffcc00, #ffaa00);
  color: #222;
  font-weight: bold;
  font-size: 1rem;
  border: none;
  border-radius: 10px;
  padding: 10px 18px;
  cursor: pointer;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.3);
  transition: all 0.15s ease;
  user-select: none;
  text-shadow: 1px 1px 0 rgba(255, 255, 255, 0.4);
  margin-bottom: 15px;
}

.upload-btn:hover {
  background: linear-gradient(180deg, #ffe066, #ffb700);
  transform: translateY(-1px);
}

.upload-btn:active {
  background: linear-gradient(180deg, #ffb700, #ff9900);
  transform: translateY(1px);
}

.compact-btn {
  background: linear-gradient(180deg, #ffdd33, #ffaa00);
  color: #222;
  font-weight: bold;
  font-size: 0.95rem;
  border: none;
  border-radius: 8px;
  padding: 8px 16px;
  cursor: pointer;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.25);
  transition: all 0.15s ease;
  margin-left: 10px;
  text-shadow: 1px 1px 0 rgba(255, 255, 255, 0.6);
}

.compact-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 10px rgba(255, 200, 0, 0.4);
}

.shine-size-label {
  font-weight: bold;
  color: #333;
  font-size: 1rem;
  text-shadow: 1px 1px 0 rgba(255, 255, 255, 0.4);
  margin-left: 10px;
  user-select: none;
}

.shine-size-select {
  background: linear-gradient(180deg, #ffdd33, #ffaa00);
  color: #222;
  font-weight: bold;
  font-size: 0.95rem;
  border: none;
  border-radius: 8px;
  padding: 8px 14px;
  cursor: pointer;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.25);
  transition: all 0.15s ease;
  margin-left: 8px;
  text-shadow: 1px 1px 0 rgba(255, 255, 255, 0.6);
  appearance: none;
}

.shine-size-select:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 10px rgba(255, 200, 0, 0.4);
}

.shine-size-select:active {
  transform: translateY(1px);
  background: linear-gradient(180deg, #ffb700, #ff9900);
}

.shine-log {
  margin-top: 2rem;
  background: rgba(20, 20, 20, 0.75);
  border: 2px solid rgba(255, 255, 255, 0.15);
  border-radius: 10px;
  padding: 12px;
  max-height: 500px;
  overflow-y: auto;
  font-family: 'JetBrains Mono', 'Courier New', monospace;
  font-size: 0.9rem;
  color: #e8e8e8;
  box-shadow: inset 0 0 8px rgba(0, 0, 0, 0.4);
}

.shine-log-title {
  color: #ffcc00;
  font-weight: bold;
  margin-bottom: 8px;
  text-shadow: 1px 1px 0 #000;
}

.shine-log-messages {
  display: flex;
  flex-direction: column;
  gap: 6px; /* spacing between entries */
}

.shine-log-entry {
  background: rgba(255, 255, 255, 0.07);
  border-radius: 6px;
  padding: 6px 8px;
  line-height: 1.3;
  border-left: 3px solid #ffaa00;
  transition: background 0.2s;
}

.shine-log-entry:hover {
  background: rgba(255, 255, 255, 0.15);
}
</style>
