
# Vue 3 Multi-slider

[![Downloads](https://img.shields.io/npm/dm/vue3-multi-slider.svg?style=flat-square)](https://www.npmjs.com/package/cropperjs) [![Version](https://img.shields.io/npm/v/vue3-multi-slider.svg?style=flat-square)](https://www.npmjs.com/package/cropperjs) [![Gzip Size](https://img.shields.io/bundlephobia/minzip/vue3-multi-slider.svg?style=flat-square)](https://unpkg.com/cropperjs/dist/cropper.common.js) [![MIT License](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)](https://choosealicense.com/licenses/mit/)


A vue 3 component for multiple sliders in one (categories)
You can use it to make distributions of percentages or other units (passed in props)

Colors, range, categories and display are customizable
## Demo

![demo_1](https://raw.githubusercontent.com/ArcenIce/vue3-multi-slider/dev/docs/gifs/demo_1.gif)
![demo_2](https://raw.githubusercontent.com/ArcenIce/vue3-multi-slider/dev/docs/gifs/demo_2.gif)


## Installation

Install vue3-multi-slider with npm or yarn

```bash
npm install vue3-multi-slider --save
```

```bash
yarn add vue3-multi-slider
```

## Usage

```html
<template>
    <MultiSlider
        v-model="distribution"
        :min="0"
        :max="100"
        :step="1"
        unit="%"
        inputs-width="200px"
        :show-inputs="true"
        :ticks="[0, 25, 50, 75, 100]"
    ></MultiSlider>
</template>
```

```javascript
<script setup lang="ts">
import { reactive } from 'vue';
import { MultiSlider } from 'vue3-multi-slider';

const distribution = reactive({
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
  }
})
</script>
```

## Props

##### min
type: number
description: minimum index value for all sliders
default: 0

##### max
type: number
description: maximum index value for all sliders
default: 100

##### step
type: number
description: values step
default: 0.01

##### unit
type: string
description: the unit displayed in inputs and slider values, it can be anything
default: %

##### inputs-width
type: string
format: css width (px or % or ...)
default: 200px
description: width for the inputs underneath the slider

##### show-inputs
type: boolean
default: true
description: hide or not the inputs underneath the slider

##### ticks
type: array
default: [min, (min + middle) / 2, middle, (middle + max) / 2, max]
description: the ticks to be displayed under the slider

## Model

To pass categories to the slider, you must define a v-model this way:
value must be reactive in order to catch updates from the component

```javascript
{
  'Category': {
      'value': 20,
      'background_color': '#478eff',
      'text_color': '#ffffff'
  },
}
```
    
## Contributing

Contributions are always welcome!

See `contributing.md` for ways to get started.

Please adhere to this project's `code of conduct`.

