<template>
    
    <div class="range-slider" ref="slider">
        <input 
        type="range" 
        ref="categories" 
        id="categories"
        :min="min"
        :max="max"
        :step="props.step || 0.01"
        v-model.number="distribution[categorie]['index']"
        :class="{'enabled-handle': enabled_handle, 'disabled-handle': !enabled_handle}"
        list="steplistmulti"
        v-for="_, categorie in distribution"
        :key="categorie"
        />
        
        <div class="slider-value" 
        :style="{ 
            'left': values_left[categorie], 
            'background-color': distribution[categorie]['background_color'], 
            'color': distribution[categorie]['text_color'] || '#ffffff'
        }" 
        v-for="_, categorie in distribution" 
        :key="categorie"
        >
        {{ (distribution[categorie]['value'] || 0).toFixed(2) }}{{ props.unit }}
        <span class="triangle" :style="{'border-top-color': distribution[categorie]['background_color']}"></span>
    </div>
    
    <div class="sliderticks"><p v-for="option in ticks_options" :key="option">{{ option }}</p></div>
    <datalist id="steplistmulti"><option v-for="option in ticks_options" :key="option" :value="option" disabled></option></datalist>
</div>

<div class="input-list" v-if="props.showInputs">
    <div v-for="_, category in distribution" :key="category">
        <div class="input-container" :style="{ width: props.inputsWidth }">
            <input
            type="number"
            id="input"
            :value="distribution[category]['value'].toFixed(2)"
            suffix="%"
            :step="props.step || 0.01"
            :min="0"
            :max="max"
            @change="event => updateCategories(category, (event.target as HTMLInputElement)?.value ? Number((event.target as HTMLInputElement).value) : 0)"
            @blur="event => updateCategories(category, (event.target as HTMLInputElement)?.value ? Number((event.target as HTMLInputElement).value) : 0)"
            :color="distribution[category]['background_color']"
            :style="{ width: props.inputsWidth || '200px' }"
            >
            <label for="input" class="label">{{ category }}</label>
            <div class="underline" :style="{ 'background-color': distribution[category]['background_color']}"></div>
            <div class="color-circle">
                <svg width="15" height="12" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                    <circle cx="50%" cy="50%" r="50" :fill="distribution[category]['background_color']"/>
                </svg>
            </div>
            <div class="unit-display">{{ props.unit }}</div>
        </div>
    </div>
</div>

</template>

<script lang="ts" setup>
import {
    ref,
    reactive,
    watch,
    computed,
    onMounted,
    onUnmounted,
    defineModel
} from 'vue';

interface DistributionItem {
    value: number;
    index?: number;
    background_color?: string;
    text_color?: string;
    obj?: HTMLElement;
}

interface Distribution {
    [key: string]: DistributionItem;
}

const props = defineProps<{
    unit: string;
    min: number;
    max: number;
    step: number;
    ticks: number[];
    inputsWidth: string;
    showInputs: boolean;
}>(); 

const emit = defineEmits(['update:modelValue']);
defineOptions({ inheritAttrs: false });

const modelDistribution = defineModel<Distribution>({ required: true });
const distribution = ref<Distribution>(modelDistribution.value);
const slider = ref<HTMLElement | null>(null);

const categories = ref<HTMLElement[]>([]);
const enabled_handle = true;
const values_left = reactive<{ [key: string]: string }>({});
const minimum_gap = 0;
const min = props.min || 0;
const max = props.max || 100;
const middle = (min + max) / 2;
const ticks_options = props.ticks || [min, (min + middle) / 2, middle, (middle + max) / 2, max];

function updateIndexesFromValue() {
    let start = min;
    for (const categorie of Object.keys(distribution.value)) {
        distribution.value[categorie]['index'] = start + distribution.value[categorie]['value'];
        start += distribution.value[categorie]['value'];
    }
}

const indexes = computed(() => {
    let start = min;
    const indexes_obj: { [key: string]: any } = {};
    for (const categorie of Object.keys(distribution.value)) {
        indexes_obj[categorie] = [start, start + distribution.value[categorie]['value']];
        start += distribution.value[categorie]['value'];
    }
    indexes_obj['restant'] = max - start;
    return indexes_obj;
});

function updateCategories(categorie: any, val: any) {
    const ecart = val - distribution.value[categorie]['value'];
    const categoryKeys = Object.keys(distribution.value);
    const index_categorie = categoryKeys.indexOf(categorie);
    
    distribution.value[categoryKeys[index_categorie]]['index']! += ecart;
}

function updateValues() {
    const keys = Object.keys(distribution.value);
    for (let i = 0; i < keys.length - 1; i++) {
        distribution.value[keys[i + 1]]['value'] =
        distribution.value[keys[i + 1]]['index']! - distribution.value[keys[i]]['index']!;
    }
    distribution.value[keys[0]]['value'] = distribution.value[keys[0]]['index']!;
}

function fillSliders() {
    const index_list = indexes.value;
    
    for (const [key, category] of Object.entries(distribution.value)) {
        if (category.obj) {
            category.obj.style.background = `
      linear-gradient(
          to right,
          rgba(0,0,0,0) 0%,
          rgba(0,0,0,0) ${(((index_list[key][0] - min) / (max - min)) * 100)}%,
          ${category.background_color} ${(((index_list[key][0] - min) / (max - min)) * 100)}%,
          ${category.background_color} ${(((index_list[key][1] - min) / (max - min)) * 100)}%, 
          rgba(0,0,0,0) ${(((index_list[key][1] - min) / (max - min)) * 100)}%, 
          rgba(0,0,0,0) 100%
      )`;
            
            if (slider.value == null) return;
            
            const newValue = Number((category.index! - min) * 100 / (max - min));
            const newPosition = -12 - (newValue * 0.2);
            
            values_left[key] = `calc(${newValue}% + (${newPosition}px))`;
        }
    }
    
    const keys = Object.keys(distribution.value);
    const last_key = keys[keys.length - 1];
    const last_obj = distribution.value[last_key];
    
    if (last_obj.obj) {
        last_obj.obj.style.background = `
      linear-gradient(
        to right,
        rgba(0,0,0,0) 0%,
        rgba(0,0,0,0) ${(((index_list[last_key][0] - min) / (max - min)) * 100)}%,
        ${last_obj.background_color} ${(((index_list[last_key][0] - min) / (max - min)) * 100)}%,
        ${last_obj.background_color} ${(((index_list[last_key][1] - min) / (max - min)) * 100)}%, 
        #e6e3e3 ${(((index_list[last_key][1] - min) / (max - min)) * 100)}%, 
        #e6e3e3 100%
    )`;
    }
}

onMounted(() => {
    updateIndexesFromValue();
    
    const cles_categories = Object.keys(distribution.value);
    
    for (let i = 0; i < categories.value.length; i++) {
        categories.value[i].style.zIndex = `${categories.value.length - i}`;
        distribution.value[cles_categories[i]].obj = categories.value[i];
        
        watch(
        () => distribution.value[cles_categories[i]].index,
        (newVal, oldVal) => {
            // Deny condition for incrementation or decrementation
            if (newVal! >= max + 0.01 || newVal! <= min - 0.01) {
                distribution.value[cles_categories[i]].index = oldVal;
            }
            
            if (newVal! > oldVal!) {
                // Moving slider right
                if (i != categories.value.length - 1) {
                    const diff = newVal! - distribution.value[cles_categories[i + 1]].index!;
                    if (diff >= -minimum_gap) {
                        distribution.value[cles_categories[i + 1]].index = newVal! + minimum_gap;
                    }
                }
            } else if (i !== 0) {
                // Moving slider left
                const diff = newVal! - distribution.value[cles_categories[i - 1]].index!;
                if (diff <= minimum_gap) {
                    distribution.value[cles_categories[i - 1]].index = newVal! - minimum_gap;
                }
            }

            updateValues();
            fillSliders();
        }
        );
    }
    
    fillSliders();
    
});

onUnmounted(() => {
    distribution.value = {};
});

watch(
distribution,
(newVal) => {
    distribution.value = { ...newVal };
    updateIndexesFromValue();
    fillSliders();
}
);
</script>


<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Poppins:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&display=swap');

* {
    font-family: "Poppins", sans-serif !important;
}

.range-slider {
    position: relative;
    min-height: 50px;
    margin-top: 60px;
    margin-bottom: 10px;
}


.slider-value {
    position: absolute;
    top: -55px;
    background-color: #272D67;
    width: 50px;
    height: 35px;
    border-radius: 0.25em;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 12px;
    font-weight: 500;
    color: white;
}

.triangle {
    display: block;
    height: 0px;
    width: 0px;
    border: 10px solid transparent;
    border-top-color: #272D67;
    position: absolute;
    bottom: -20px;
    left: calc(50% - 10px);
}

input[type=range] {
    appearance: none;
    -webkit-appearance: none;
}

input[type=range].disabled-handle::-webkit-slider-thumb {
    pointer-events: none;
    visibility: hidden;
}

input[type=range].disabled-handle::-moz-range-thumb {
    pointer-events: none;
    visibility: hidden;
}

input[type=range].enabled-handle::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    pointer-events: all;
    background-color: #ffffff;
    border-radius: 15px;
    height: 20px;
    width: 20px;
    margin-top: -6px;
    transition: all 0.3s;
    filter: drop-shadow(0px 0px 3px #cfcfcf);
    cursor: pointer;
}

input[type=range].enabled-handle::-moz-range-thumb {
    pointer-events: all;
    background-color: #ffffff;
    height: 20px;
    width: 20px;
    border-radius: 50%;
    transition: all 0.3s;
    filter: drop-shadow(0px 0px 3px #cfcfcf);
    cursor: pointer;
}

input[type=range].enabled-handle::-ms-thumb {
    pointer-events: all;
    background-color: #ffffff;
    border-radius: 15px;
    height: 20px;
    width: 20px;
    transition: all 0.3s;
    filter: drop-shadow(0px 0px 3px #cfcfcf);
    cursor: pointer;
}

.sliderticks {
    display: flex;
    justify-content: space-between;
    padding: 20px 12px;
    margin-bottom: -10px;
    pointer-events: none;
}

.sliderticks p {
    position: relative;
    display: flex;
    justify-content: center;
    text-align: center;
    width: 1px;
    background: #D3D3D3;
    color: #8b8b8b;
    height: 10px;
    line-height: 40px;
    margin: 0 0 0px 0;
    pointer-events: none;
    user-select: none;
    font-size: 12px;
}

input[type=range]::-webkit-slider-thumb:hover {
    transform: scale(1.1);
}

input[type=range]::-moz-range-thumb:hover {
    transform: scale(1.1);
}

input[type=range]::-webkit-slider-thumb:active {
    transform: scale(1.2);
}

input[type=range]::-moz-range-thumb:active {
    transform: scale(1.2);
}

input[type="range"] {
    -webkit-appearance: none; 
    appearance: none;
    height: 15px;
    width: 100%;
    border-radius: 10px;
    position: absolute;
    background-color: #C6C6C6;
    pointer-events: none;
}

input[type=number]::-webkit-inner-spin-button, 
input[type=number]::-webkit-outer-spin-button { 
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
    margin: 0; 
}


.input-list {
    width: 100%;
    display: flex;
    flex-wrap: wrap;
    justify-content: left;
    gap: 18px;
    padding: 0 5px;
}


/* From Uiverse.io by Satwinder04 */ 
.input-container {
    position: relative;
    margin-top: 20px;
}

.input-container input[type="number"] {
    font-size: 14px;
    width: 100%;
    border: none;
    border-bottom: 2px solid #ccc;
    padding: 5px 0;
    background-color: transparent;
    outline: none;
    box-sizing: border-box;
    padding-left: 24px;
}

.input-container .label {
    position: absolute;
    left: 0px;
    color: #ccc;
    transition: all 0.3s ease;
    pointer-events: none;
}

.input-container input[type="number"]:focus ~ .label,
.input-container input[type="number"]:valid ~ .label {
    top: -16px;
    font-size: 12px;
    color: #b2b2b2;
}

input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
    -webkit-appearance: none;
    margin: 0;
}

/* Firefox */
input[type=number] {
    appearance: textfield;
    -moz-appearance: textfield;
}

.input-container .underline {
    pointer-events: none;
    position: absolute;
    bottom: 0;
    left: 0;
    height: 2px;
    width: 100%;
    background-color: #b2b2b2;
    transform: scaleX(0);
    transition: all 0.3s ease;
}

.input-container .color-circle {
    position: absolute;
    pointer-events: none;
    bottom: 5px;
    left: 4px;
    width: 100%;
    transition: all 0.3s ease;
}

.input-container .unit-display {
    position: absolute;
    pointer-events: none;
    bottom: 5px;
    right: 0px;
    transition: all 0.3s ease;
}

.input-container input[type="number"]:focus ~ .underline {
    transform: scaleX(1);
    background-color: #333;
}


</style>