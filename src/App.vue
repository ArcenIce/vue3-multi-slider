<template>
  <div style="width: 50dvw;"> 
    <MultiSlider
      v-model="model"
      :min="0"
      :max="100"
      :step="1"
      unit="%"
      inputs-width="200px"
      :show-inputs="true"
      :ticks="[0, 25, 50, 75, 100]"
    ></MultiSlider>
  </div>
</template>

<script setup lang="ts">
import { reactive } from 'vue';
import MultiSlider from './components/MultiSlider.vue';

const model = reactive({
  'Category 1': {
      'value': 20,
      'background_color': '#478eff',
      'text_color': '#ffffff'
  },
  'Category 2': {
      'value': 20,
      'background_color': '#86f7a8',
      'text_color': '#ffffff'
  },
  'Category 3': {
      'value': 20,
      'background_color': '#86f0f7',
      'text_color': '#00000'
  },
  'Category 4': {
      'value': 20,
      'background_color': '#8FFFFF',
      'text_color': '#00000'
  }
})
</script>
