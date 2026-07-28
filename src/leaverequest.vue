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
            <h2>Leave Request Preview</h2>

            <button
                class="modal-close"
                @click="showPreview = false"
            >
                ✕
            </button>
            </div>

            <div class="modal-body">

            <div class="preview-item">
                <strong>Employee Name:</strong>
                {{ form.employeeName }}
            </div>

            <div class="preview-item">
                <strong>Department:</strong>
                {{ form.department }}
            </div>

            <div class="preview-item">
                <strong>Leave Type:</strong>
                {{ form.leaveType }}
            </div>

            <div class="preview-item">
                <strong>Start Date:</strong>
                {{ form.startDate }}
            </div>

            <div class="preview-item">
                <strong>End Date:</strong>
                {{ form.endDate }}
            </div>

            <div class="preview-item">
                <strong>Reason:</strong>
                <p>{{ form.reason }}</p>
            </div>

            <div class="preview-item">
                <strong>Attachments:</strong>

                <ul v-if="attachments.length">
                <li
                    v-for="file in attachments"
                    :key="file.name"
                >
                    {{ file.name }}
                </li>
                </ul>

                <span v-else>No attachment</span>
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
                Confirm & Submit
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
  width: 70%;            
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

.button-group {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 24px;
}

.preview-item {
  margin-bottom: 18px;
}

.preview-item strong {
  display: block;
  margin-bottom: 4px;
}

.preview-item p {
  margin: 6px 0 0;
}

.modal {
  max-width: 700px;
}
</style>