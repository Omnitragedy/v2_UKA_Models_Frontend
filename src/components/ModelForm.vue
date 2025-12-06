<script setup>
import { reactive, toRefs, watch, computed } from 'vue'
import VariableField from './VariableField.vue'

const props = defineProps({
  fields: {
    type: Array,
    required: true
  },
  initialValues: {
    type: Object,
    default: () => ({})
  }
})

const emit = defineEmits(['submit', 'reset', 'update:model'])

// initialize model with field keys and sensible defaults
function createModelFromFields(fields, initial = {}) {
  const m = {}
  for (const f of fields) {
    if (initial && Object.prototype.hasOwnProperty.call(initial, f.key)) {
      m[f.key] = initial[f.key]
      continue
    }
    // sensible defaults by type
    switch (f.type) {
      case 'number':
        m[f.key] = f.default ?? null
        break
      case 'checkbox':
        m[f.key] = f.default ?? false
        break
      case 'multiselect':
        m[f.key] = f.default ?? []
        break
      default:
        m[f.key] = f.default ?? ''
    }
  }
  return m
}

const model = reactive(createModelFromFields(props.fields, props.initialValues))

// keep model in sync if fields or initialValues change
watch(
  () => [props.fields, props.initialValues],
  () => {
    const next = createModelFromFields(props.fields, props.initialValues)
    for (const k of Object.keys(next)) model[k] = next[k]
    // remove deleted keys
    for (const k of Object.keys(model)) {
      if (!Object.prototype.hasOwnProperty.call(next, k)) delete model[k]
    }
  },
  { deep: true }
)

function updateField(key, value) {
  model[key] = value
  emit('update:model', { ...model })
}

function handleSubmit() {
  console.log("handleSubmit()")
  emit('submit', { ...model })
}

function handleReset() {
  const next = createModelFromFields(props.fields, props.initialValues)
  for (const k of Object.keys(next)) model[k] = next[k]
  emit('reset')
  emit('update:model', { ...model })
}

const groupedFields = computed(() => {
  // group fields by input-type categories: "Options" (all fields with predefined options)
  // and "Inputs" (free-text / numeric / textarea)
  const groups = {}

  function typeGroup(f) {
    const t = f.type
    // include multiselects together with select/radio so all predefined-option fields are grouped the same
    if ((t === 'select' || t === 'radio' || t === 'multiselect') && Array.isArray(f.options)) {
      // treat any field with options (including yes/no) as an "Options" field
      return 'Options'
    }
    // numbers, textareas and other radios without options are considered Inputs
    return 'Inputs'
  }

  for (const f of props.fields) {
    const g = typeGroup(f)
    if (!groups[g]) groups[g] = []
    groups[g].push(f)
  }
  return groups
})

const orderedGroups = computed(() => {
  // render options first, then inputs
  const preferred = ['Options', 'Inputs']
  const groups = groupedFields.value
  const entries = []

  for (const name of preferred) {
    if (groups[name]) entries.push([name, groups[name]])
  }

  // append any remaining groups in original order
  for (const [k, v] of Object.entries(groups)) {
    if (!preferred.includes(k)) entries.push([k, v])
  }

  return entries
})
</script>


<template>
  <form @submit.prevent="handleSubmit" class="model-form">
    <div v-for="(entry) in orderedGroups" :key="entry[0]" class="field-group">
      <div class="group-fields">
        <VariableField
          v-for="field in entry[1]"
          :key="field.key"
          :field="field"
          :modelValue="model[field.key]"
          @update:modelValue="val => updateField(field.key, val)"
        />
      </div>
    </div>

    <div class="form-actions">
      <button type="button" class="btn btn-secondary" @click="handleReset">Reset</button>
      <button type="submit" class="btn btn-primary" @click="console.log('🟢 BUTTON CLICKED')">
        Run Model
      </button>
    </div>
  </form>
</template>

<style scoped>
.model-form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
.field-group {
  border-top: 1px dashed rgba(16,24,40,0.05);
  padding-top: 0.75rem;
}
.group-title {
  font-size: 0.95rem;
  margin: 0 0 0.5rem;
  color: #0f172a;
  font-weight: 700;
}
.group-fields {
  display: flex;
  flex-direction: column;
}
.form-actions {
  display: flex;
  gap: 0.6rem;
  justify-content: flex-end;
  margin-top: 0.25rem;
}
.btn {
  padding: 0.5rem 0.9rem;
  border-radius: 8px;
  border: 1px solid transparent;
  cursor: pointer;
  font-weight: 600;
}
.btn-primary {
  background: #2563eb;
  color: #fff;
  border-color: #2563eb;
}
.btn-secondary {
  background: #fff;
  color: #0f172a;
  border-color: #e6e9ef;
}
</style>
