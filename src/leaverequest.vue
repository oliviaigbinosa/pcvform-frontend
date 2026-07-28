<template>
  <div class="page-header">
    <div class="page-header__left">

      <div class="header-icon">
        <svg
          width="26"
          height="26"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
        >
          <rect x="3" y="7" width="18" height="14" rx="2" />
          <path d="M8 7V5a4 4 0 0 1 8 0v2" />
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

        <div class="employee-name-row">
        <FormField
            label="Employee Name"
            v-model="form.employeeName"
            placeholder="Enter your full name"
        />
        </div>

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
            class="btn btn-outline"
            @click="showPreview = true"
        >
            Preview
        </button>

        <button
            class="btn btn-primary"
            type="submit"
        >
            Submit Request
        </button>
        </div>
  </form>
       <div
  v-if="showPreview"
  class="modal-backdrop"
  @click.self="showPreview = false"
>
  <div class="modal">

    <div class="modal-header">
      <div class="modal-header__title">
        <span class="preview-icon">👁</span>
        Form Preview
      </div>

      <button
        class="modal-close"
        @click="showPreview = false"
      >
        ✕
      </button>
    </div>

    <div class="modal-body">

      <div class="preview-head">
        <div>
          <p class="company-label">
            Getpayed Technology Solutions Ltd.
          </p>

          <h2 class="preview-title serif">
            Leave Request Form
          </h2>
        </div>

        <div class="preview-head__right">
          <p class="mono-label tiny">
            Request No.
          </p>

          <code class="voucher-no">
            LRF/2026/07/001
          </code>
        </div>
      </div>

      <div class="preview-section">
        <h3 class="preview-section__title mono-label">
          Employee Details
        </h3>

        <div class="preview-rows">
          <div class="preview-row">
            <span>Employee Name</span>
            <span>{{ form.employeeName || "—" }}</span>
          </div>

          <div class="preview-row">
            <span>Department</span>
            <span>{{ form.department || "—" }}</span>
          </div>
        </div>
      </div>

      <div class="preview-section">
        <h3 class="preview-section__title mono-label">
          Leave Details
        </h3>

        <div class="preview-rows">
          <div class="preview-row">
            <span>Leave Type</span>
            <span>{{ form.leaveType || "—" }}</span>
          </div>

          <div class="preview-row">
            <span>Start Date</span>
            <span>{{ form.startDate || "—" }}</span>
          </div>

          <div class="preview-row">
            <span>End Date</span>
            <span>{{ form.endDate || "—" }}</span>
          </div>

          <div class="preview-row block-row">
            <span>Reason</span>
            <span>{{ form.reason || "—" }}</span>
          </div>
        </div>
      </div>

      <div class="preview-section">
        <h3 class="preview-section__title mono-label">
          Supporting Documents
        </h3>

        <div class="preview-rows">
          <div class="preview-row">
            <span>Attached Files</span>

            <span>
              <template v-if="attachments.length">
                <span
                  v-for="(file, i) in attachments"
                  :key="i"
                >
                  {{ file.name }}{{ i < attachments.length - 1 ? ", " : "" }}
                </span>
              </template>

              <template v-else>
                None
              </template>
            </span>
          </div>
        </div>
      </div>

    </div>

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
        Confirm &amp; Submit
      </button>
    </div>

  </div>
</div>
</template>

<script setup>
import { reactive } from 'vue'
import FormField from '../components/FormField.vue'
import FileUpload from '../components/FileUpload.vue'
import { ref } from 'vue'

const showPreview = ref(false)
const form = reactive({
  employeeName: '',
  employeeId: '',
  department: '',
  leaveType: '',
  startDate: '',
  endDate: '',
  reason: '',
})
const attachments = reactive([])

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
  margin: 20px auto 28px;
}

.page-header__left {
  display: flex;
  align-items: center;
  gap: 18px;
}

.employee-name-row {
  grid-column: 1 / -1;   
  width: 100%;            
}
.header-text {
  padding-top: 4px;
}

.header-icon {
  width: 46px;
  height: 46px;
  background: #203864;
  border-radius: 12px;

  display: flex;
  align-items: center;
  justify-content: center;

  color: #fff;
  flex-shrink: 0;

  margin-top: -4px;
}

.company-label {
  font-size: 11px;
  letter-spacing: 0.22em;
  text-transform: uppercase;
  color: #8b7355;

  margin-top: 6px;
  margin-bottom: 4px;
}

.serif {
  font-family: Georgia, serif;
  font-size: 30px;
  font-weight: 700;
  line-height: 1.05;
  margin: 2px 0 0;
}

.page-subtitle {
  margin-top: 6px;
  color: #6b7280;
  font-size: 15px;
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
  max-width: 760px;
  width: 100%;
  border-radius: 8px;
  overflow: hidden;
  background: #fff;
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px 28px;
  border-bottom: 1px solid #eee;
}

.modal-header__title {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 16px;
  font-weight: 600;
  color: #1a2b4c;
}

.modal-body {
  padding: 28px;
  max-height: 70vh;
  overflow-y: auto;
}

/* ===== Header ===== */

.preview-head {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding-bottom: 18px;
  border-bottom: 1px solid #ececec;
  margin-bottom: 24px;
}

.preview-head__right {
  text-align: right;
}

.company-label {
  font-size: 11px;
  letter-spacing: .22em;
  text-transform: uppercase;
  color: #8b7355;
  margin: 0 0 6px;
}

.preview-title {
  margin: 0;
  color: #1a2b4c;
}

.serif {
  font-family: Georgia, serif;
  font-size: 28px;
  font-weight: 700;
}

.mono-label {
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: .18em;
  color: #8a92a0;
}

.tiny {
  margin-bottom: 6px;
}

.voucher-no {
  background: #f4f4f4;
  padding: 6px 10px;
  border-radius: 6px;
  font-size: 13px;
  color: #1a2b4c;
}

/* ===== Sections ===== */

.preview-section {
  margin-top: 24px;
}

.preview-section:first-of-type {
  margin-top: 0;
}

.preview-section__title {
  padding-bottom: 8px;
  margin-bottom: 14px;
  border-bottom: 1px solid #ececec;
}

/* ===== Rows ===== */

.preview-rows {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.preview-row {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 24px;
  font-size: 14px;
}

.preview-row span:first-child {
  color: #8a92a0;
  font-weight: 500;
}

.preview-row span:last-child {
  color: #1a2b4c;
  text-align: right;
}

.block-row {
  flex-direction: column;
  gap: 6px;
}

.block-row span:last-child {
  text-align: left;
}

/* ===== Footer ===== */

.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  padding: 18px 28px;
  border-top: 1px solid #eee;
}

</style>