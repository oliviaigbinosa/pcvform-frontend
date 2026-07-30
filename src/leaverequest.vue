<template>
  <div class="content">

    <div class="page-header">

      <div class="page-header__left">

        <div class="header-icon">
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

        <div class="header-text">
          <p class="company-label">
            Getpayed Technology Solutions Ltd.
          </p>

          <h1 class="serif">
            Leave Request Form
          </h1>
        </div>

      </div>

    </div>

    <form class="card" @submit.prevent="submitLeave">

      <div class="form-grid">

        <FormField
  label="Employee Name"
  v-model="form.employeeName"
  placeholder="Enter your full name"
/>

<FormField
  label="Department Manager"
  v-model="form.departmentManager"
  placeholder="Enter department manager"
/>

        <FormField
          type="select"
          label="Department"
          v-model="form.department"
          :options="departmentOptions"
          placeholder="Select Department"
        />

        <FormField
          type="select"
          label="Leave Type"
          v-model="form.leaveType"
          :options="leaveOptions"
          placeholder="Select leave type"
        />

        <FormField
          type="date"
          label="Start Date"
          v-model="form.startDate"
        />

        <FormField
          type="date"
          label="End Date"
          v-model="form.endDate"
        />

        <FormField
          class="full-width"
          type="textarea"
          label="Reason"
          v-model="form.reason"
          placeholder="State the reason for your leave request"
        />
                <FileUpload
                class="full-width"
                label="Supporting Documents (Optional)"
                v-model:files="attachments"
                />
      </div>

      <div class="button-group">
  <button
  type="button"
  class="btn btn-primary"
  @click="showPreview = true"
>
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

    </form>
     <div
  v-if="showPreview"
  class="modal-backdrop"
  @click.self="showPreview = false"
>
  <div class="modal">

    <!-- Modal Header -->
    <div class="modal-header">
      <div class="modal-header__title">
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

        Form Preview
      </div>

      <button
        class="modal-close"
        @click="showPreview = false"
      >
        ✕
      </button>
    </div>

    <!-- Modal Body -->
    <div class="modal-body">

  <div class="preview-title-block">

    <p class="company-label">
      Getpayed Technology Solutions Ltd.
    </p>

    <h1 class="preview-title serif">
      Leave Request Form
    </h1>

  </div>

  <hr class="preview-divider" />
  
      <div class="preview-row">
  <span class="preview-label">Employee Name</span>
  <span class="preview-value">{{ form.employeeName || "—" }}</span>
</div>

<div class="preview-row">
  <span class="preview-label">Department Manager</span>
  <span class="preview-value">{{ form.departmentManager || "—" }}</span>
</div>
      

      <div class="preview-row">
        <span class="preview-label">Department</span>
        <span class="preview-value">{{ form.department || "—" }}</span>
      </div>

      <div class="preview-row">
        <span class="preview-label">Leave Type</span>
        <span class="preview-value">{{ form.leaveType || "—" }}</span>
      </div>

      <div class="preview-row">
        <span class="preview-label">Start Date</span>
        <span class="preview-value">{{ form.startDate || "—" }}</span>
      </div>

      <div class="preview-row">
        <span class="preview-label">End Date</span>
        <span class="preview-value">{{ form.endDate || "—" }}</span>
      </div>

      <div class="preview-row preview-row--block">
        <span class="preview-label">Reason</span>
        <span class="preview-value">
          {{ form.reason || "—" }}
        </span>
      </div>

      <div class="preview-row preview-row--block">
        <span class="preview-label">Attachments</span>

        <span class="preview-value" v-if="attachments.length">
  <ul class="preview-file-list">
    <li
      v-for="(file, i) in attachments"
      :key="i"
    >
      <a
        href="#"
        class="file-link"
        @click.prevent="openFilePreview(file)"
      >
        {{ file.name }}
      </a>
    </li>
  </ul>
</span>
        <span
          class="preview-value"
          v-else
        >
          None
        </span>
      </div>

    </div>

    <!-- Modal Footer -->
    <div class="modal-footer">

      <button
        class="btn btn-outline"
        @click="showPreview = false"
      >
        Back
      </button>

      <button
        class="btn btn-primary"
        @click="submitLeave"
      >
        Submit Form
      </button>
      </div>

    </div>
  </div>
</div>
 
        <FilePreview
        :show="showFilePreview"
        :file="previewFile"
        @close="showFilePreview = false"
        />
</template>

<script setup>
import FilePreview from '../components/FilePreview.vue'
import { reactive } from 'vue'
import FormField from '../components/FormField.vue'
import FileUpload from '../components/FileUpload.vue'
import { ref } from 'vue'

const showPreview = ref(false)
const showFilePreview = ref(false)
const previewFile = ref(null)
const form = reactive({
  employeeName: '',
  departmentManager: "",
  department: '',
  leaveType: '',
  startDate: '',
  endDate: '',
  reason: '',
})
const attachments = ref([])

const leaveOptions = [
  { label: 'Annual Leave', value: 'annual' },
  { label: 'Sick Leave', value: 'sick' },
  { label: 'Casual Leave', value: 'casual' },
  { label: 'Maternity Leave', value: 'maternity' },
  { label: 'Paternity Leave', value: 'paternity' },
]

const departmentOptions = [
  { label: 'Human Resources', value: 'HR' },
  { label: 'Information Technology', value: 'IT' },
  { label: 'Finance', value: 'Finance' },
  { label: 'Operations', value: 'Operations' },
  { label: 'Marketing', value: 'Marketing' },
]

function openFilePreview(file) {
  previewFile.value = file
  showFilePreview.value = true
}

function submitLeave() {
  console.log(form)
  console.log(attachments)
}
</script>

<style scoped>
.page {
  max-width: 950px;
  margin: 0 auto;
  padding: 40px 0;
}

.page-header {
  width: 750px;
  margin: 20px auto 20px;
}

.employee-name-row {
  grid-column: 1 / -1;   
  width: 100%;            
}

.page-header__left {
  display: flex;
  align-items: flex-start;
  gap: 14px;
}


.header-icon {
  width: 44px;
  height: 44px;
  border-radius: 10px;
  background: var(--primary);
  color: var(--primary-fg);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.company-label {
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  color: var(--muted-fg);
  margin: 0 0 4px 0;
}

.serif {
  font-family: var(--font-serif);
  font-size: 22px;
  font-weight: 700;
  line-height: 1.2;
  margin: 0;
}

.page-subtitle {
  margin-top: 8px;
  color: var(--muted-fg);
  font-size: 14px;
}


.card {
  width: 100%;
  max-width: 750px;
  margin: 0 auto;
  background: #fff;
  border-radius: 22px;
  padding: 34px;
  box-sizing: border-box;
}

.card {
  margin-bottom: 40px;
}

.content {
  max-width: 750px;
  margin: 0 auto;
  padding-top: 24px;
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 28px;
}

.full-width {
  grid-column: 1 / -1;
}


.btn {
  margin-top: 28px;
}


@media (max-width: 768px) {

  .page {
    padding: 24px;
  }

  .page-header__left {
    gap: 14px;
  }

  .header-icon {
    width: 42px;
    height: 42px;
    margin-top: 4px;
  }

  .serif {
    font-size: 32px;
  }

  .form-grid {
    grid-template-columns: 1fr;
  }

  .card {
    padding: 24px;
  }
}

.modal {
  background: var(--card);
  border-radius: var(--radius-lg);
  border: 1px solid var(--border);
  width: 100%;
  max-width: 640px;
  box-shadow: 0 24px 60px rgba(0, 0, 0, 0.18);
  margin: 0 auto;
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 24px;
  border-bottom: 1px solid var(--border);
}

.modal-header__title {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 14px;
  font-weight: 600;
  color: var(--primary);
}

.modal-close {
  width: 32px;
  height: 32px;
  border-radius: 8px;
  border: none;
  background: transparent;
  cursor: pointer;
  color: var(--muted-fg);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  transition:
    background 0.15s,
    color 0.15s;
}

.modal-close:hover {
  background: var(--muted);
  color: var(--fg);
}

.modal-body {
  padding: 24px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.modal-footer {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 12px;
  padding: 16px 24px;
  border-top: 1px solid var(--border);
}

/* ── Preview header ─────────────────────────────────────────── */
.preview-head {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  padding-bottom: 24px;
  margin-bottom: 24px;
  border-bottom: 1px solid var(--border);
}

.preview-title {
  font-family: var(--font-serif);
  font-size: 20px;
  font-weight: 700;
  margin-top: 4px;
}

.preview-head__right {
  text-align: right;
}

/* ── Preview sections ───────────────────────────────────────── */
.preview-section {
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding: 12px 0;
}

.preview-section__title {
  padding-bottom: 8px;
  border-bottom: 1px solid var(--border);
}

.preview-rows {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.preview-row {
  display: flex;
  align-items: flex-start;
  gap: 20px;
  padding: 4px 0;
  font-size: 14px;
  line-height: 1.6;
}

.preview-row > span:first-child {
  flex-shrink: 0;
  width: 160px;
  color: var(--muted-fg);
  font-size: 11px;              /* same as company label */
  font-weight: 500;
  letter-spacing: 0.04em;
  text-transform: uppercase;    /* makes them uppercase */
}

.preview-row > span:last-child {
  flex: 1;
  color: var(--fg);
  word-break: break-word;
}

.preview-row.block-row {
  flex-direction: column;
  gap: 8px;
}

.preview-row.block-row > span:first-child {
  width: auto;
}

.button-group {
  display: flex;
  justify-content: flex-end;
  margin-top: 28px;
}

.review-btn {
  width: 100%;
  justify-content: center;
  padding: 14px;
  font-size: 15px;
  margin-top: 4px;
}
</style>