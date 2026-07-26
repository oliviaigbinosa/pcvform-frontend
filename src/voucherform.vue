<template>
  <!-- ════════════════ FORM VIEW ════════════════ -->
  <div class="page-wrap">
    <div v-if="submitted" class="card success-card">
      <div class="success-icon">✓</div>
      <h2 class="serif">Voucher Submitted</h2>
      <p>Your voucher has been sent by email.</p>
      <code class="voucher-badge">{{ lastVoucherNo }}</code>
      <div class="success-actions">
        <button class="btn btn-outline" @click="router.push({ name: 'vouchers' })">
          View My Vouchers
        </button>
        <button class="btn btn-primary" @click="resetForm">New Voucher</button>
      </div>
    </div>

    <div v-else class="container">
      <header class="pcv-header">
        <div class="pcv-header__left">
          <div class="pcv-logo">
            <svg
              width="20"
              height="20"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
            >
              <rect x="2" y="7" width="20" height="14" rx="2" />
              <path d="M16 7V5a2 2 0 0 0-2-2h-4a2 2 0 0 0-2 2v2" />
            </svg>
          </div>
          <div>
            <p class="company-label">Getpayed Technology Solutions Ltd.</p>
            <h1 class="serif">Petty Cash Voucher</h1>
          </div>
        </div>
        <div class="pcv-header__right">
          <p class="mono-label">Voucher No.</p>
          <code class="voucher-no">{{ voucherNoDisplay }}</code>
        </div>
      </header>

      <nav class="stepper" aria-label="Form steps">
        <div
          v-for="(label, i) in STEPS"
          :key="i"
          class="stepper__item"
          :class="{ done: i < step, active: i === step }"
        >
          <div class="stepper__bubble">
            <svg
              v-if="i < step"
              width="14"
              height="14"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="3"
            >
              <polyline points="20 6 9 17 4 12" />
            </svg>
            <span v-else>{{ i + 1 }}</span>
          </div>
          <span class="stepper__label">{{ label }}</span>
        </div>
      </nav>

      <div class="card step-card">
        <!-- Step 0 -->
        <template v-if="step === 0">
          <div class="step-head">
            <svg
              width="16"
              height="16"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
            >
              <line x1="22" y1="2" x2="11" y2="13" />
              <polygon points="22 2 15 22 11 13 2 9 22 2" />
            </svg>
            <h2>Email Details</h2>
          </div>
          <div class="fields">
            <FormField
              v-model="form.from"
              label="From (Sender Email)"
              type="email"
              placeholder="your-name@getpayedmail.com"
              readonly
            />
            <div ref="toDropdownRef" class="field custom-select">
              <label class="mono-label">To (Recipient Email)</label>
              <button
                type="button"
                class="custom-select__trigger"
                :class="{ error: errors.to }"
                @click.stop="toDropdownOpen = !toDropdownOpen"
              >
                <span class="custom-select__label">{{ form.to || 'Select recipient' }}</span>
                <svg
                  class="custom-select__chevron"
                  width="14"
                  height="14"
                  viewBox="0 0 24 24"
                  fill="none"
                  stroke="currentColor"
                  stroke-width="3"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                >
                  <polyline points="6 9 12 15 18 9" />
                </svg>
              </button>
              <div v-if="toDropdownOpen" class="custom-select__options">
                <button
                  type="button"
                  class="custom-select__option"
                  :class="{ selected: form.to === '' }"
                  @click="selectTo('')"
                >
                  Select recipient
                </button>
                <button
                  v-for="user in onboardingUsers"
                  :key="user.email"
                  type="button"
                  class="custom-select__option"
                  :class="{ selected: form.to === user.email }"
                  @click="selectTo(user.email)"
                >
                  {{ user.email }}
                </button>
              </div>
              <span v-if="errors.to" class="err-msg">{{ errors.to }}</span>
            </div>
            <FormField
              v-model="form.cc"
              label="CC (notified on approval)"
              type="email"
              placeholder="finance@getpayedmail.com"
              readonly
            />
            <FormField
              v-model="form.subject"
              label="Subject"
              type="text"
              filter="[^A-Za-z\s]"
              placeholder="Petty Cash Request — Department — Month Year"
              :error="errors.subject"
              @input="clearErr('subject')"
            />
          </div>
        </template>

        <!-- Step 1 -->
        <template v-else-if="step === 1">
          <div class="step-head">
            <svg
              width="16"
              height="16"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
            >
              <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z" />
              <polyline points="14 2 14 8 20 8" />
            </svg>
            <h2>Payee Information</h2>
          </div>
          <div class="fields">
            <FormField
              v-model="form.payee"
              label="Payee (Person / Vendor)"
              type="text"
              filter="[^A-Za-z\s.,;:!?'()&-]"
              placeholder="e.g. John Adeyemi / Staples Ltd."
              :error="errors.payee"
              @input="clearErr('payee')"
            />
            <FormField
              v-model="form.department"
              label="Department"
              type="text"
              filter="[^A-Za-z\s]"
              placeholder="e.g. Marketing, Operations, HR"
              :error="errors.department"
              hint="Populates the voucher number above."
              @input="clearErr('department')"
            />
          </div>
        </template>

        <!-- Step 2 -->
        <template v-else-if="step === 2">
          <div class="step-head">
            <h2>Amount &amp; Purpose</h2>
          </div>
          <div class="fields">
            <FormField
              v-model="displayAmount"
              label="Amount in Figures (NGN)"
              prefix="₦"
              inputmode="decimal"
              placeholder="0,000.00"
              input-class="mono-input"
              :error="errors.amountFigures"
              @input="clearErr('amountFigures')"
            />
            <FormField
              v-model="form.amountWords"
              label="Amount in Words"
              type="textarea"
              :rows="2"
              readonly
            />
            <FormField
              v-model="form.purpose"
              label="Purpose / Description"
              type="textarea"
              :rows="4"
              filter="[^A-Za-z\s.,;:!?'()&-]"
              placeholder="Describe what this petty cash is for..."
              :error="errors.purpose"
              @input="clearErr('purpose')"
            />
          </div>
        </template>

        <!-- Step 3 -->
        <template v-else-if="step === 3">
          <div class="step-head">
            <svg
              width="16"
              height="16"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
            >
              <polyline points="16 16 12 12 8 16" />
              <line x1="12" y1="12" x2="12" y2="21" />
              <path d="M20.39 18.39A5 5 0 0 0 18 9h-1.26A8 8 0 1 0 3 16.3" />
            </svg>
            <h2>Supporting Documents</h2>
          </div>
          <div class="fields">
            <FormField
              v-model="form.submissionDate"
              label="Submission Date"
              type="date"
              readonly
              :error="errors.submissionDate"
              @change="clearErr('submissionDate')"
            />
            <FileUpload
              v-model:files="form.supportingDocs"
              label="Attach Supporting Documents"
              upload-title="Click to attach files"
              hint="Files are listed in the email body."
              @preview="openFilePreview"
            />
            <button type="button" class="btn btn-accent review-btn" @click="openPreview">
              <svg
                width="16"
                height="16"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
              >
                <path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z" />
                <circle cx="12" cy="12" r="3" />
              </svg>
              Review Form
            </button>
          </div>
        </template>
      </div>

      <div class="nav-row">
        <button class="btn btn-outline" :disabled="step === 0" @click="back">
          <svg
            width="15"
            height="15"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
          >
            <polyline points="15 18 9 12 15 6" />
          </svg>
          Back
        </button>
        <span class="step-counter mono-label">Step {{ step + 1 }} of {{ STEPS.length }}</span>
        <button v-if="step < STEPS.length - 1" class="btn btn-primary" @click="next">
          Next
          <svg
            width="15"
            height="15"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
          >
            <polyline points="9 18 15 12 9 6" />
          </svg>
        </button>
        <button v-else class="btn btn-primary" :disabled="loadingVouchers" @click="openPreview">
          <svg
            width="15"
            height="15"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
          >
            <path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z" />
            <circle cx="12" cy="12" r="3" />
          </svg>
          Review &amp; Send
        </button>
      </div>
    </div>

    <FormPreview
      v-model="showPreview"
      :form="form"
      :voucher-no="voucherNo"
      :parsed-amount="parsedAmount"
      :formatted-amount="formattedAmount"
      :user-email="userEmail"
      @submit="submitVoucher"
    />
    <FilePreview
      :show="showFilePreview"
      :file="previewFile"
      @close="showFilePreview = false"
    />
  </div>
</template>

<script setup>
import { ref, reactive, computed, watch, onMounted, onBeforeUnmount } from 'vue'
import { useRouter } from 'vue-router'
import FormField from '../components/FormField.vue'
import FileUpload from '../components/FileUpload.vue'
import FormPreview from '../components/formpreview.vue'
import FilePreview from '../components/FilePreview.vue'
import { addVoucher, userEmail, onboardingUsers, fetchOnboardingUsers, allVouchers, loadingVouchers } from './stores/appState'

// ─── Constants ────────────────────────────────────────────────────────────────
const STEPS = ['Email Details', 'Payee Info', 'Amount & Purpose', 'Documents & Review']

// ─── Number → words ───────────────────────────────────────────────────────────
const ones = [
  '',
  'One',
  'Two',
  'Three',
  'Four',
  'Five',
  'Six',
  'Seven',
  'Eight',
  'Nine',
  'Ten',
  'Eleven',
  'Twelve',
  'Thirteen',
  'Fourteen',
  'Fifteen',
  'Sixteen',
  'Seventeen',
  'Eighteen',
  'Nineteen',
]
const tensArr = [
  '',
  '',
  'Twenty',
  'Thirty',
  'Forty',
  'Fifty',
  'Sixty',
  'Seventy',
  'Eighty',
  'Ninety',
]

function belowThousand(n, includeAnd = false) {
  if (n === 0) return ''
  if (n < 20) return ones[n]
  if (n < 100) {
    const tens = tensArr[Math.floor(n / 10)]
    const rest = n % 10 ? `-${ones[n % 10]}` : ''
    return tens + rest
  }

  const hundredPart = `${ones[Math.floor(n / 100)]} Hundred`
  const remainder = n % 100
  if (remainder === 0) return hundredPart
  return includeAnd
    ? `${hundredPart} and ${belowThousand(remainder)}`
    : `${hundredPart} ${belowThousand(remainder)}`
}

function numberToWords(amount) {
  if (isNaN(amount) || amount <= 0) return ''

  const naira = Math.floor(amount)
  const kobo = Math.round((amount - naira) * 100)
  const hasNaira = naira > 0
  const hasKobo = kobo > 0

  if (!hasNaira && hasKobo) {
    return `${belowThousand(kobo).trim()} Kobo Only`
  }

  const scales = [
    { value: 1_000_000_000_000, label: 'Trillion' },
    { value: 1_000_000_000, label: 'Billion' },
    { value: 1_000_000, label: 'Million' },
    { value: 1_000, label: 'Thousand' },
  ]

  let remaining = naira
  const chunks = []

  for (const scale of scales) {
    const value = Math.floor(remaining / scale.value)
    if (value > 0) {
      chunks.push({ val: value, label: scale.label })
    }
    remaining %= scale.value
  }

  if (remaining > 0) {
    chunks.push({ val: remaining, label: '' })
  }

  let words = chunks
    .filter((c) => c.val > 0)
    .map((c) => {
      const chunkWords = belowThousand(c.val, true).trim()
      return chunkWords ? `${chunkWords} ${c.label}`.trim() : ''
    })
    .filter(Boolean)
    .join(', ')
    .trim()

  words += ' Naira'
  if (hasKobo) words += ' and ' + belowThousand(kobo).trim() + ' Kobo'
  return words + ' Only'
}

// ─── Helpers ──────────────────────────────────────────────────────────────────
function pad(n) {
  return String(n).padStart(2, '0')
}
function formatNumberWithCommas(num) {
  return num.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 })
}
function normalizeAmount(val) {
  const raw = String(val ?? '').replace(/[^0-9.]/g, '')
  const parts = raw.split('.')
  if (parts.length === 1) return parts[0]
  return `${parts[0]}.${parts.slice(1).join('').slice(0, 2)}`
}
function formatNumberInput(val) {
  const normalized = normalizeAmount(val)
  if (normalized === '') return ''
  const dotIndex = normalized.indexOf('.')
  if (dotIndex === -1) {
    return parseInt(normalized || '0', 10).toLocaleString('en-US')
  }
  const int = normalized.slice(0, dotIndex)
  const dec = normalized.slice(dotIndex + 1)
  const formattedInt = parseInt(int || '0', 10).toLocaleString('en-US')
  return `${formattedInt}.${dec}`
}
function currentYear() {
  return new Date().getFullYear()
}
function currentMonth() {
  return pad(new Date().getMonth() + 1)
}
function monthName(m) {
  return (
    ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'][
      parseInt(m) - 1
    ] ?? m
  )
}
function todayStr() {
  const d = new Date()
  return `${d.getFullYear()}-${pad(d.getMonth() + 1)}-${pad(d.getDate())}`
}
function isEmail(v) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v)
}

const router = useRouter()
const step = ref(0)
const showPreview = ref(false)
const submitted = ref(false)
const lastVoucherNo = ref('')
const showFilePreview = ref(false)
const previewFile = ref(null)
const toDropdownOpen = ref(false)
const toDropdownRef = ref(null)

onMounted(async () => {
  try {
    await fetchOnboardingUsers()
  } catch {
    // Silently fail - users can still manually enter email if needed
  }
  window.addEventListener('click', closeToDropdown)
})

onBeforeUnmount(() => {
  window.removeEventListener('click', closeToDropdown)
})

const form = reactive({
  from: userEmail.value,
  to: '',
  cc: 'finance@getpayedmail.com',
  subject: '',
  payee: '',
  department: '',
  amountFigures: '',
  amountWords: '',
  purpose: '',
  supportingDocs: [],
  submissionDate: todayStr(),
})

const errors = reactive({})

const deptSlug = computed(() => form.department.trim().toUpperCase().replace(/\s+/g, '-') || 'DEPT')
const voucherNo = computed(() => {
  const prefix = `PCV/${deptSlug.value}/${currentYear()}/${currentMonth()}`
  const used = allVouchers.value
    .filter((v) => v.id?.startsWith(`${prefix}/`))
    .map((v) => {
      const serial = v.id.split('/').pop() || '0'
      const n = parseInt(serial, 10)
      return isNaN(n) ? 0 : n
    })
  const max = used.length ? Math.max(...used) : 0
  return `${prefix}/${String(max + 1).padStart(3, '0')}`
})

const voucherNoDisplay = computed(() => (loadingVouchers.value ? '—' : voucherNo.value))
const parsedAmount = computed(() => parseFloat(form.amountFigures) || 0)
const formattedAmount = computed(() => formatNumberWithCommas(parsedAmount.value))
const displayAmount = computed({
  get() {
    return formatNumberInput(form.amountFigures)
  },
  set(value) {
    form.amountFigures = normalizeAmount(value)
  },
})

watch(
  () => form.department,
  (dept) => {
    if (!form.subject || form.subject.startsWith('Petty Cash Request')) {
      form.subject = `Petty Cash Request — ${dept || 'Department'} — ${monthName(currentMonth())} ${currentYear()}`
    }
  },
)
watch(
  () => form.amountFigures,
  (val) => {
    form.amountWords = numberToWords(parseFloat(val))
  },
)

const validators = {
  0() {
    const e = {}
    if (!isEmail(form.to)) {
      e.to = 'Valid recipient email required'
    } else if (!form.to.endsWith('@getpayedmail.com') && form.to !== 'igbinosaolivia6@gmail.com') {
      e.to = 'Email must be a @getpayedmail.com address or igbinosaolivia6@gmail.com'
    }
    if (!form.subject.trim()) e.subject = 'Subject is required'
    return e
  },
  1() {
    const e = {}
    if (!form.payee.trim()) e.payee = 'Payee name is required'
    if (!form.department.trim()) e.department = 'Department is required'
    return e
  },
  2() {
    const e = {}
    const n = parseFloat(form.amountFigures)
    if (!form.amountFigures || isNaN(n) || n <= 0) e.amountFigures = 'Enter a valid amount'
    if (!form.purpose.trim()) e.purpose = 'Purpose is required'
    return e
  },
  3() {
    const e = {}
    if (!form.submissionDate) e.submissionDate = 'Submission date is required'
    return e
  },
}

function validate() {
  const errs = validators[step.value]?.() ?? {}
  Object.assign(errors, errs)
  Object.keys(errors).forEach((k) => {
    if (!errs[k]) delete errors[k]
  })
  return Object.keys(errs).length === 0
}

function clearErr(key) {
  delete errors[key]
}

function selectTo(email) {
  form.to = email
  clearErr('to')
  toDropdownOpen.value = false
}

function closeToDropdown(event) {
  if (toDropdownRef.value && !toDropdownRef.value.contains(event.target)) {
    toDropdownOpen.value = false
  }
}
function next() {
  if (validate()) step.value = Math.min(step.value + 1, STEPS.length - 1)
}
function back() {
  Object.keys(errors).forEach((k) => delete errors[k])
  step.value = Math.max(step.value - 1, 0)
}
function openPreview() {
  if (validate()) showPreview.value = true
}
function openFilePreview(file) {
  previewFile.value = file
  showFilePreview.value = true
}
async function submitVoucher() {
  const entry = {
    id: voucherNo.value,
    submittedBy: userEmail.value,
    submissionDate: form.submissionDate,
    payee: form.payee,
    department: form.department,
    amount: parsedAmount.value,
    amountWords: form.amountWords,
    purpose: form.purpose,
    from: form.from,
    to: form.to,
    cc: form.cc,
    subject: form.subject,
    supportingDocs: form.supportingDocs.map((f) => ({
      name: f.name,
      type: f.type,
      size: f.size,
      data: f.data,
    })),
  }

  lastVoucherNo.value = voucherNo.value
  showPreview.value = false
  submitted.value = true
  await addVoucher(entry)
}

function resetForm() {
  Object.assign(form, {
    from: userEmail.value,
    to: '',
    cc: 'finance@getpayedmail.com',
    subject: '',
    payee: '',
    department: '',
    amountFigures: '',
    amountWords: '',
    purpose: '',
    supportingDocs: [],
    submissionDate: todayStr(),
  })
  step.value = 0
  submitted.value = false
  Object.keys(errors).forEach((k) => delete errors[k])
}
</script>
