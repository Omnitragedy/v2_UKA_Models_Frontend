\
<script setup>
// const API_BASE = import.meta.env.DEV
//   ? "http://127.0.0.1:8000"
//   : "https://surgery-prediction-api--predict.modal.run"

const API_BASE = 'https://omnitragedy--surgery-prediction-api-modelservice-predict.modal.run'

import { ref, onMounted, reactive } from 'vue'
import ColumnLayout from './components/ColumnLayout.vue'
import ModelForm from './components/ModelForm.vue'
import valueMap from './assets/unique_values_mapping.json'

function titleCase(str) {
  return String(str)
    .toLowerCase()
    .split(/[\s\-_\/]+/)
    .map(s => s.charAt(0).toUpperCase() + s.slice(1))
    .join(' ')
}

onMounted(async () => {
  try {
    await fetch(`https://omnitragedy--surgery-prediction-api-modelservice-warmup.modal.run`, {
      method: "POST",
      headers: { "Content-Type": "application/json" }
    });
    console.log("✅ Backend warmed (models loaded, no inference run)");
  } catch (e) {
    console.warn("⚠️ Warmup failed", e);
  }
});

const apiStatusSD = ref({
  visible: false,
  code: null,
  message: '',
  type: 'info'
})

const apiStatusLOS = ref({
  visible: false,
  code: null,
  message: '',
  type: 'info'
})

const variableDescriptions = {
  ANESTHES: 'Principal Anesthesia Technique',
  PRPTT: 'Pre-Operative PTT',
  PRINR: 'Pre-Operative INR',
  AGE: 'Age',
  PRCREAT: 'Pre-Operative Serum Creatinine',
  FNSTATUS2: 'Functional Status Prior to Surgery',
  PRALBUM: 'Pre-Operative Serum Albumin',
  PRSGOT: 'Pre-Operative SGOT',
  BLEEDDIS: 'Bleeding Disorders',
  SEX: 'Sex',
  PRHCT: 'Pre-Operative Hematocrit',
  ASACLAS: 'ASA Class',
  PRBUN: 'Pre-Operative BUN',
  DIABETES: 'DM (Taking Oral Agents or Insulin)',
  WEIGHT: 'Weight',
  ETOH: '>2 Drinks/Day'
}

 // Fields for Surgical Duration (SD) regression model
const fieldsSD = [
  {
    key: 'SEX',
    label: 'Sex',
    type: 'radio',
    group: 'Demographics',
    options: [
      { value: 'male', label: 'Male' },
      { value: 'female', label: 'Female' },
      { value: 'other', label: 'Other' }
    ],
    help: 'Sex'
  },
  {
    key: 'ANESTHES',
    label: 'Principal Anesthesia Technique',
    type: 'radio',
    group: 'Procedure',
    options: valueMap.ANESTHES.map(v => ({ value: v, label: titleCase(v) })),
    help: 'Principal Anesthesia Technique'
  },
  {
    key: 'ASACLAS',
    label: 'ASA Class',
    type: 'radio',
    group: 'History',
    options: valueMap.ASACLAS.map(v => ({ value: v, label: titleCase(v) })),
    help: 'ASA Class',
    default: valueMap.ASACLAS[0],
    required: true
  },
  {
    key: 'AGE',
    label: 'Age',
    type: 'number',
    group: 'Demographics',
    min: 0,
    max: 150,
    placeholder: 'Years',
    help: 'Age'
  },
  {
    key: 'PRPTT',
    label: 'Pre-Operative PTT',
    type: 'number',
    group: 'Labs',
    min: 0,
    step: 0.1,
    help: 'Pre-Operative PTT'
  },
  {
    key: 'PRINR',
    label: 'Pre-Operative INR',
    type: 'number',
    group: 'Labs',
    min: 0,
    step: 0.01,
    help: 'Pre-Operative INR'
  },
  {
    key: 'PRCREAT',
    label: 'Pre-Operative Serum Creatinine',
    type: 'number',
    group: 'Labs',
    min: 0,
    step: 0.01,
    help: 'Pre-Operative Serum Creatinine'
  },
  {
    key: 'FNSTATUS2',
    label: 'Functional Status Prior to Surgery',
    type: 'radio',
    group: 'History',
    options: valueMap.FNSTATUS2.map(v => ({ value: v, label: titleCase(v) })),
    help: 'Functional Status Prior to Surgery',
    default: "Independent",
    required: true
  },
  {
    key: 'PRALBUM',
    label: 'Pre-Operative Serum Albumin',
    type: 'number',
    group: 'Labs',
    step: 0.1,
    help: 'Pre-Operative Serum Albumin'
  },
  {
    key: 'PRSGOT',
    label: 'Pre-Operative SGOT',
    type: 'number',
    group: 'Labs',
    step: 0.1,
    help: 'Pre-Operative SGOT'
  },
  {
    key: 'BLEEDDIS',
    label: 'Bleeding Disorders',
    type: 'radio',
    group: 'History',
    options: valueMap.BLEEDDIS.map(v => ({ value: v, label: titleCase(v) })),
    help: 'Bleeding Disorders'
  },
  {
    key: 'PRHCT',
    label: 'Pre-Operative Hematocrit',
    type: 'number',
    group: 'Labs',
    step: 0.1,
    help: 'Pre-Operative Hematocrit'
  }
]

 // Fields for Length of Stay (LOS) classification model (predict >1 day vs <=1 day)
const fieldsLOS = [
  {
    key: 'SEX',
    label: 'Sex',
    type: 'radio',
    group: 'Demographics',
    options: [
      { value: 'male', label: 'Male' },
      { value: 'female', label: 'Female' },
      { value: 'other', label: 'Other' }
    ],
    help: 'Sex'
  },
  {
    key: 'ANESTHES',
    label: 'Principal Anesthesia Technique',
    type: 'radio',
    group: 'Procedure',
    options: valueMap.ANESTHES.map(v => ({ value: v, label: titleCase(v) })),
    help: 'Principal Anesthesia Technique'
  },
  {
    key: 'AGE',
    label: 'Age',
    type: 'number',
    group: 'Demographics',
    min: 0,
    max: 150,
    placeholder: 'Years',
    help: 'Age'
  },
  {
    key: 'WEIGHT',
    label: 'Weight',
    type: 'number',
    group: 'Vitals',
    step: 0.1,
    placeholder: 'lbs',
    help: 'Weight (lbs)'
  },
  {
    key: 'PRINR',
    label: 'Pre-Operative INR',
    type: 'number',
    group: 'Labs',
    min: 0,
    step: 0.01,
    help: 'Pre-Operative INR'
  },
  {
    key: 'PRPTT',
    label: 'Pre-Operative PTT',
    type: 'number',
    group: 'Labs',
    min: 0,
    step: 0.1,
    help: 'Pre-Operative PTT'
  },
  {
    key: 'PRHCT',
    label: 'Pre-Operative Hematocrit',
    type: 'number',
    group: 'Labs',
    step: 0.1,
    help: 'Pre-Operative Hematocrit'
  },
  {
    key: 'PRBUN',
    label: 'Pre-Operative BUN',
    type: 'number',
    group: 'Labs',
    step: 0.1,
    help: 'Pre-Operative BUN'
  },
  {
    key: 'PRCREAT',
    label: 'Pre-Operative Serum Creatinine',
    type: 'number',
    group: 'Labs',
    min: 0,
    step: 0.01,
    help: 'Pre-Operative Serum Creatinine'
  },
  {
    key: 'ETOH',
    label: '>2 Drinks/Day',
    type: 'radio',
    group: 'History',
    options: valueMap.ETOH.map(v => ({ value: v, label: titleCase(v) })),
    help: 'More than 2 drinks/day'
  },
  {
    key: 'DIABETES',
    label: 'DM (Taking Oral Agents or Insulin)',
    type: 'radio',
    group: 'History',
    options: valueMap.DIABETES.map(v => ({ value: v, label: titleCase(v) })),
    help: 'Diabetes (taking oral agents or insulin)'
  },
  {
    key: 'BLEEDDIS',
    label: 'Bleeding Disorders',
    type: 'radio',
    group: 'History',
    options: valueMap.BLEEDDIS.map(v => ({ value: v, label: titleCase(v) })),
    help: 'Bleeding Disorders'
  }
]

// Reactive state for SD model
const lastResultSD = ref(null)
const statusSD = ref('idle')
const lastRunAtSD = ref(null)

// Reactive state for LOS model
const lastResultLOS = ref(null)
const statusLOS = ref('idle')
const lastRunAtLOS = ref(null)

function validateCategories(model, valueMap) {
  for (const key in model) {
    // Only validate known categorical fields
    if (!valueMap[key]) continue;

    const val = model[key];

    // ALLOW: null, undefined, empty string
    // These mean "not provided"
    if (val === null || val === undefined || val === '') {
      continue;
    }

    // REJECT: any non-null value not in dictionary
    if (!valueMap[key].includes(val)) {
      throw new Error(
        `Invalid categorical value for ${key}: ${val}. Allowed values: ${valueMap[key].join(', ')}`
      );
    }
  }
}

// Submit handler for SD regression model
async function onSubmitSD(model) {
  statusSD.value = 'running'
  lastRunAtSD.value = new Date().toISOString()

  const payload = { ...model }

  if (payload.SEX === 'other') payload.SEX = null
  if (payload.ASACLAS === 'None assigned') payload.ASACLAS = '2-Mild Disturb'

  try {
    validateCategories(payload, valueMap)

    apiStatusSD.value = {
      visible: true,
      code: "Sending...",
      message: "",
      type: "info"
    }

    const response = await fetch(API_BASE, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        model_type: "sd",
        features: payload
      })
    })

    const data = await response.json().catch(() => ({}))

    apiStatusSD.value = {
      visible: true,
      code: response.status,
      message: response.ok ? "Success" : data.error || "Backend error",
      type: response.ok ? "success" : "error"
    }

    if (!response.ok) throw new Error(data.error || "Request failed")

    lastResultSD.value = data
    statusSD.value = "done"

  } catch (error) {
    apiStatusSD.value = {
      visible: true,
      code: "ERROR",
      message: error.message,
      type: "error"
    }

    statusSD.value = "error"
    lastResultSD.value = { error: error.message }
  }
}

// Submit handler for LOS classification model
async function onSubmitLOS(model) {
  statusLOS.value = 'running'
  lastRunAtLOS.value = new Date().toISOString()

  const payload = { ...model }
  if (payload.SEX === 'other') payload.SEX = null

  try {
    validateCategories(payload, valueMap)

    apiStatusLOS.value = {
      visible: true,
      code: "Sending...",
      message: "",
      type: "info"
    }

    const response = await fetch(API_BASE, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        model_type: "los",
        features: payload
      })
    })

    const data = await response.json().catch(() => ({}))

    apiStatusLOS.value = {
      visible: true,
      code: response.status,
      message: response.ok ? "Success" : data.error || "Backend error",
      type: response.ok ? "success" : "error"
    }

    if (!response.ok) throw new Error(data.error || "Request failed")

    lastResultLOS.value = data
    statusLOS.value = "done"

  } catch (error) {
    apiStatusLOS.value = {
      visible: true,
      code: "ERROR",
      message: error.message,
      type: "error"
    }

    statusLOS.value = "error"
    lastResultLOS.value = { error: error.message }
  }
}


function onResetSD() {
  lastResultSD.value = null
  statusSD.value = 'idle'
  lastRunAtSD.value = null
}

function onResetLOS() {
  lastResultLOS.value = null
  statusLOS.value = 'idle'
  lastRunAtLOS.value = null
}
</script>

<template>
  <main class="app-main">
    <header class="app-header">
      <div class="app-header-inner">
        <h1 class="app-title">Surgery Prediction Dashboard</h1>
        <p class="app-subtitle">
          Surgical Duration & Length of Stay Forecasting with Explainable AI
        </p>
        <p class="app-subtitle">
          Fill out more inputs for more informative predictions and SHAP plots
        </p>
      </div>
    </header>

    <ColumnLayout>
      <template #left>
        <section>
          <h2 class="section-title">Surgical Duration (minutes)</h2>
          <p class="muted">Regression model — predicts expected surgical duration in minutes.</p>

          <ModelForm
            :fields="fieldsSD"
            :disabled="statusSD === 'running'"
            @submit="onSubmitSD"
            @reset="onResetSD"
            @update:model="() => {}"
          />

          <div v-if="apiStatusSD.visible"
               class="api-status-panel"
               :class="apiStatusSD.type">

            <template v-if="apiStatusSD.code === 'Sending...'">
              <span class="spinner"></span>
              <span class="status-text">Sending...</span>
            </template>

            <template v-else>
              <strong>Status:</strong> {{ apiStatusSD.code }}
              <span v-if="apiStatusSD.message"> — {{ apiStatusSD.message }}</span>
            </template>
          </div>

          <div class="result-panel" style="margin-top:1rem">
            <div class="meta"><strong>Status:</strong> <span class="status">{{ statusSD }}</span></div>

            <div class="result-box" v-if="lastResultSD">
              <h3>Prediction</h3>
              <p><strong>Duration (minutes):</strong> {{ lastResultSD.prediction.duration_minutes }}</p>
              <div v-if="lastResultSD.shap?.plot_base64" style="margin-top: 1rem;">
                <h4>SHAP Explanation of SD</h4>
                <img
                  :src="'data:image/png;base64,' + lastResultSD.shap.plot_base64"
                  alt="SHAP Waterfall SD"
                  style="max-width: 100%; border-radius: 8px; border: 1px solid #ddd;"
                />
              </div>
            </div>

            <div v-else class="empty muted">
              No SD result yet. Run the SD model from the form above.
            </div>
          </div>
        </section>
      </template>

      <template #right>
        <section>
          <h2 class="section-title">Length of Stay (> 1 day?)</h2>
          <p class="muted">Classification model — predicts whether length of stay will exceed 1 day.</p>

          <ModelForm
            :fields="fieldsLOS"
            :disabled="statusLOS === 'running'"
            @submit="onSubmitLOS"
            @reset="onResetLOS"
            @update:model="() => {}"
          />

          <div v-if="apiStatusLOS.visible"
               class="api-status-panel"
               :class="apiStatusLOS.type">
            <strong>Status:</strong> {{ apiStatusLOS.code }}
            <span v-if="apiStatusLOS.message"> — {{ apiStatusLOS.message }}</span>
          </div>


          <div class="result-panel" style="margin-top:1rem">
            <div class="meta"><strong>Status:</strong> <span class="status">{{ statusLOS }}</span></div>

            <div class="result-box" v-if="lastResultLOS">
              <h3>Prediction</h3>
              <p>
                <strong>Predicted:</strong>
                {{ lastResultLOS.prediction.over_one_day ? 'Over 1 day' : 'Under or equal to 1 day' }}
              </p>
              <p><strong>Probability(over 1 day):</strong> {{ lastResultLOS.prediction.probability_over_one_day }}</p>
              <p><strong>‎ ‎ ‎ ‎ ‎(P > 0.5 → Over 1 Day)</strong></p>
              <p><strong>‎ ‎ ‎ ‎ ‎(P < 0.5 → Under 1 Day)</strong></p>
              <div v-if="lastResultLOS.shap?.plot_base64" style="margin-top: 1rem;">
                <h4>SHAP Explanation of LOS Probability</h4>
                <img
                  :src="'data:image/png;base64,' + lastResultLOS.shap.plot_base64"
                  alt="SHAP Waterfall LOS"
                  style="max-width: 100%; border-radius: 8px; border: 1px solid #ddd;"
                />
              </div>
            </div>

            <div v-else class="empty muted">
              No LOS result yet. Run the LOS model from the form above.
            </div>
          </div>
        </section>
      </template>
    </ColumnLayout>
  </main>
</template>

<style scoped>
body {
  margin: 0;
  font-family: ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont;
  background: linear-gradient(180deg, #f8fafc, #e2e6f1);
  color: #0f172a;
}

/* Header */
.app-header {
  background: linear-gradient(135deg, #4377ec, #031d75);
  padding: 1.25rem 1rem;
  border-bottom: 1px solid rgba(255,255,255,0.15);
}

.app-header-inner {
  max-width: 1200px;
  margin: 0 auto;
}

.app-title {
  margin: 0;
  font-size: 1.6rem;
  font-weight: 800;
  color: #ffffff;
  letter-spacing: 0.3px;
}

.app-subtitle {
  margin-top: 0.3rem;
  font-size: 0.95rem;
  color: #e0e7ff;
}

/* Main Layout */
.app-main {
  margin: 1.75rem auto;
  padding: 0 0rem;
}

.app-card {
  background: white;
  border-radius: 18px;
  padding: 1.5rem;
  box-shadow:
    0 10px 20px rgba(0,0,0,0.08),
    0 2px 6px rgba(0,0,0,0.04);
}

/* Section Headers for SD & LOS */
.section-title {
  font-size: 1.1rem;
  font-weight: 700;
  color: #1e293b;
  margin-bottom: 0.6rem;
  border-bottom: 2px solid #e5e7eb;
  padding-bottom: 0.35rem;
}

/* Column Layout */
.model-columns {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem;
  margin-top: 1rem;
}

@media (max-width: 900px) {
  .model-columns {
    grid-template-columns: 1fr;
  }
}
.api-status-panel {
  margin-top: 12px;
  padding: 8px 12px;
  border-radius: 6px;
  font-size: 13px;
  font-family: monospace;
  width: fit-content;
}

.api-status-panel.success {
  background: #e8f7ee;
  color: #1b7f43;
  border: 1px solid #9fe0b8;
}

.api-status-panel.error {
  background: #fde8e8;
  color: #9b1c1c;
  border: 1px solid #f8b4b4;
}

.api-status-panel.info {
  background: #eef2ff;
  color: #3730a3;
  border: 1px solid #c7d2fe;
}

.spinner {
  width: 16px;
  height: 16px;
  border: 2.5px solid rgba(0,0,0,0.15);
  border-top-color: currentColor;
  border-radius: 50%;
  display: inline-block;
  animation: spin 0.8s linear infinite;
  vertical-align: middle;
  margin-right: 8px;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.status-text {
  vertical-align: middle;
}
</style>
