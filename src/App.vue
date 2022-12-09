<script setup>
import { ref } from 'vue'
import InfiniteKnob from './components/InfiniteKnob.vue'

const value = ref(0)
function infiniteKnob_hndNewValue(newValue) {
  value.value = Math.max(0, value.value + Math.round(newValue * 10))
} 
</script>

<template>
  <artivle>
    <h1>InfiniteKnob</h1>
    <InfiniteKnob :divisions="24" @new-value="infiniteKnob_hndNewValue" />
    <h3>Value: {{ value }}</h3>
  </artivle>
</template>
