<script setup lang="ts">

import { ref, watch, computed } from 'vue'

const options = ref("")
const selections = ref<Selection>({})
const bubble = ref(false)
const peek = ref(false)

interface Selection {
  [key: string]: string
}

interface Score {
  [key: string]: [string]
}

interface Button {
  selection: string,
  button0: string,
  button1: string,
  choice: string
}

watch(options, (newOptions, oldOptions) => {
  if (!newOptions) {
    selections.value = {}
  } else {
    let newPairStrings = arrayToPairStrings(optionsToList(newOptions))
    let oldPairStrings = arrayToPairStrings(optionsToList(oldOptions))
    let newSelections: Selection = {}
    for (let pairString of oldPairStrings) {
      if (newPairStrings.indexOf(pairString) > -1) {
        newSelections[pairString] = selections.value[pairString]
      }
    }
    for (let pairString of newPairStrings) {
      if (oldPairStrings.indexOf(pairString) === -1) {
        newSelections[pairString] = ""
      }
    }
    selections.value = newSelections
  }
})

const sortedOptions = computed(() => {
  let sOptions:Score = {};
  for (let option of optionsToList(options.value)) {
    let score = Object.values(selections.value).filter(s => s === option).length.toString()
    if (score in sOptions) {
      sOptions[score].push(option)
    } else {
      sOptions[score] = [option]
    }
  }
  // return Object.entries(sOptions).toSorted().toReversed()
  return Object.entries(sOptions).sort().reverse()
})

const allChosen = computed(() => {
  return Object.keys(selections.value).length && Object.values(selections.value).every(s => s !== "")
})

const selectionsLength = computed(() => {
  return Object.keys(selections.value).length
})

const numberRemaining = computed(() => {
  return Object.values(selections.value).filter(s => s === "").length
})
        
const buttons = computed<Button[]>(() => {
  let buttons: Button[] = []
  for (let selection of Object.entries(selections.value)) {
    let choices = JSON.parse(selection[0])
    buttons.push({
      "selection": selection[0],
      "button0": choices[0],
      "button1": choices[1],
      "choice": selection[1]
    })
  }
  if (bubble.value) {
    return buttons.sort(buttonSort)
  } else {
    return buttons.sort(defaultSort)
  }
})

function arrayToPairStrings(ary: string[]) {
  let pairs = [];
  for (let i = 0 ; i < ary.length - 1 ; i++) {
    for (let j = i + 1 ; j < ary.length ; j++) {
      pairs.push(JSON.stringify([ary[i], ary[j]]))
    }
  }
  return pairs;
}

function choose(selection: string, idx: number) {
  let pair = JSON.parse(selection)
  selections.value[selection] = pair[idx]
}

function optionsToList(o: string) {
  return [...new Set(o.split("\n").filter((item: string) => item !== ""))]
}

function buttonSort(a: Button, b: Button) {
  if (a.choice === "" && b.choice !== "") {
    return -1
  } else if (a.choice !== "" && b.choice === "") {
    return 1
  } else {
    return defaultSort(a, b)
  }
}

function defaultSort(a: Button, b: Button) {
  let optionsList = optionsToList(options.value)
  let firstOptionComparison = optionsList.indexOf(a.button0) - optionsList.indexOf(b.button0)
  if (firstOptionComparison !== 0) {
    return firstOptionComparison
  } else {
    return optionsList.indexOf(a.button1) - optionsList.indexOf(b.button1)
  }
}

</script>

<template>
        <h1>Priorities</h1>
	<div class="grid">
	  <div id="docs">
	    <details aria-label="explanation">
	      <summary>This is a tool for prioritizing a list of items.</summary>
	      <p>Sorting a list of three options is easy to do off the top of your head, but it's less obvious how to order ten options. This tool orders them by counting the number of times each option was chosen in all of the pairwise comparisons; for <math>n</math> options, the number of pairs is the sum of the integers from <math>n - 1</math> down to <math>1</math>. In the case of ten options, you'd make 45 comparisons.</p>
	      <details aria-label="settings">
		<summary><h2>⚙</h2></summary>
		<p><button :class="{ secondary: !bubble }" @click="bubble = !bubble">bubble</button> <i>Unmade choices float to the top</i></p>
		<p><button :class="{ secondary: !peek }" @click="peek = !peek">peek</button> <i>Show scores before completion</i></p>
	      </details>
	    </details>
	  </div>
	</div>
	<div class="grid">
          <div id="options">
            <p>
              <i>Type or paste options here, one to a line</i>
            </p>
            <textarea cols="30" id="" name="" rows="10" v-model="options">
            </textarea>
          </div>
          <div id="selections">
            <p v-if="selectionsLength"><i>Choose preferences ({{ numberRemaining }} of {{ selectionsLength }} remaining)</i></p>
            <article v-for="pair of buttons" :key="pair.button0 + pair.button1">
              <button :class="{ secondary: pair.button0 !== pair.choice }" @click="choose(pair.selection, 0)">{{ pair.button0 }}</button> or <button :class="{ secondary: pair.button1 !== pair.choice }" @click="choose(pair.selection, 1)">{{ pair.button1 }}</button>
            </article>
          </div>
          <div>
            <div v-if="allChosen || (peek && options)" id="sortedoptions">
              <p><i>Scored options</i></p>
              <template v-for="score of sortedOptions" :key="score">
                <p>
	          <i>{{ score[0] }}</i>: <span v-for="(option, idx) in score[1]" :key="idx"><span v-if="idx != 0">&nbsp;</span><mark>{{ option }}</mark></span>
	        </p>
               </template>
            </div>
          </div>
        </div>
</template>

<style scoped>
</style>
