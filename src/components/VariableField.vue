<script setup>
import { computed } from 'vue'
import variableDescriptions from '../data/variable_descriptions.js'

const props = defineProps({
  field: {
    type: Object,
    required: true
  },
  modelValue: {
    type: [String, Number, Boolean, Array, Object],
    default: ''
  }
})

const emit = defineEmits(['update:modelValue'])

// normalize value emitted from native inputs
function onInput(event) {
  const t = props.field.type
  if (t === 'number') {
    const n = event.target.valueAsNumber
    emit('update:modelValue', Number.isNaN(n) ? null : n)
  } else if (t === 'checkbox') {
    emit('update:modelValue', event.target.checked)
  } else if (t === 'multiselect') {
    const selected = Array.from(event.target.selectedOptions).map(o => o.value)
    emit('update:modelValue', selected)
  } else {
    emit('update:modelValue', event.target.value)
  }
}

const inputId = computed(() => `field-${props.field.key}`)

const helpText = computed(() => {
  // show the original variable name (same as the tooltip) as the help text
  // keep fallback to explicit help or descriptions if key is missing (shouldn't happen)
  return props.field.key || props.field.help || variableDescriptions[props.field.key] || ''
})

function clearSelection() {
  // clear the current value for option-style controls (radios/selects)
  // emits an empty string which matches the default model initialization strategy
  emit('update:modelValue', '')
}
</script>

<template>
  <div class="variable-field" :class="field.type">
    <label :for="inputId" class="vf-label" :title="field.key">
      <span class="vf-title">{{ field.label }}</span>
      <span v-if="helpText" class="vf-help"> — {{ helpText }}</span>
    </label>

    <div class="vf-control">
      <template v-if="field.type === 'text'">
        <input
          :id="inputId"
          type="text"
          :placeholder="field.placeholder || ''"
          :value="modelValue"
          @input="onInput"
        />
      </template>

      <template v-else-if="field.type === 'number'">
        <input
          :id="inputId"
          type="number"
          :placeholder="field.placeholder || ''"
          :value="modelValue"
          @input="onInput"
          :min="field.min"
          :max="field.max"
          :step="field.step || 'any'"
        />
      </template>

      <template v-else-if="field.type === 'textarea'">
        <textarea
          :id="inputId"
          :placeholder="field.placeholder || ''"
          :value="modelValue"
          @input="onInput"
          rows="3"
        />
      </template>

      <template v-else-if="field.type === 'select'">
        <select :id="inputId" :value="modelValue" @change="onInput" :required="field.required">
          <option value="" disabled hidden v-if="field.placeholder">{{ field.placeholder }}</option>
          <option v-for="opt in field.options" :key="opt.value" :value="opt.value">
            {{ opt.label }}
          </option>
        </select>
      </template>

      <template v-else-if="field.type === 'multiselect'">
        <select :id="inputId" multiple @change="onInput">
          <option v-for="opt in field.options" :key="opt.value" :value="opt.value">
            {{ opt.label }}
          </option>
        </select>
      </template>

      <template v-else-if="field.type === 'checkbox'">
        <input
          :id="inputId"
          type="checkbox"
          :checked="!!modelValue"
          @change="onInput"
        />
      </template>

      <template v-else-if="field.type === 'radio'">
        <div class="radio-group">
          <label v-for="opt in field.options" :key="opt.value" class="radio-option">
            <input
              type="radio"
              :name="field.key"
              :value="opt.value"
              :checked="modelValue === opt.value"
              @change="onInput"
              :required="field.required"
            />
            {{ opt.label }}
          </label>

          <!-- small clear button to remove radio selection (hidden for required fields like ASACLAS) -->
          <button
            v-if="modelValue !== '' && modelValue !== null && modelValue !== undefined && !field.required"
            type="button"
            class="vf-clear"
            @click="clearSelection"
            :title="`Clear ${field.key} selection`"
          >✕</button>
        </div>
      </template>

      <template v-else>
        <!-- fallback to text -->
        <input
          :id="inputId"
          type="text"
          :value="modelValue"
          @input="onInput"
        />
      </template>
    </div>

    <div v-if="field.error" class="vf-error">{{ field.error }}</div>
  </div>
</template>

<style scoped>
.variable-field {
  margin-bottom: 0.9rem;
  display: flex;
  flex-direction: column;
}
.vf-label {
  font-weight: 600;
  margin-bottom: 0.35rem;
  font-size: 0.95rem;
}
.vf-help {
  font-weight: 400;
  color: #666;
  font-size: 0.9rem;
}
.vf-control input[type="text"],
.vf-control input[type="number"],
.vf-control textarea,
.vf-control select {
  width: 100%;
  padding: 0.45rem 0.6rem;
  border: 1px solid #d0d4db;
  border-radius: 6px;
  background: white;
  box-sizing: border-box;
  font-size: 0.95rem;
}
.vf-control input[type="checkbox"] {
  transform: scale(1.05);
}
.radio-group {
  display: flex;
  gap: 0.8rem;
  flex-wrap: wrap;
}
.radio-option {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  font-size: 0.95rem;
}

/* clear button for radio groups */
.vf-clear {
  margin-left: 0.5rem;
  background: transparent;
  border: 1px solid rgba(16,24,40,0.06);
  padding: 0.15rem 0.35rem;
  border-radius: 6px;
  cursor: pointer;
  color: #374151;
  font-size: 0.9rem;
  line-height: 1;
}
.vf-clear:hover {
  background: rgba(16,24,40,0.03);
}

/* error message */
.vf-error {
  color: #b00020;
  margin-top: 0.35rem;
  font-size: 0.85rem;
}
</style>
