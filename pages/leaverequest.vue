<template>
  <div class="content">

    <div class="page-header">

      <div class="page-header__left">

        <div v-if="activeTab !== 'requests'" class="header-icon">
          <svg
              width="20"
              height="20"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
            >
              <rect x="3" y="4" width="18" height="18" rx="2" />
              <path d="M16 2v4M8 2v4M3 10h18" />
              <path d="M9 15l2 2 4-4" />
            </svg>
        </div>

        <div class="header-text">
          <p v-if="activeTab !== 'requests'" class="company-label">
            Getpayed Technology Solutions Ltd.
          </p>

          <h1 class="serif">
            {{ activeTab === 'requests' ? 'Leave Requests' : 'Leave Request Form' }}
          </h1>

          <p v-if="activeTab === 'requests'" class="vouchers-sub">
            {{ displayedLeaveRequests.length }} leave request{{ displayedLeaveRequests.length !== 1 ? 's' : '' }} submitted by users
          </p>
        </div>

      </div>

    </div>

    <nav v-if="isAdmin" class="dashboard-tabs" role="tablist" aria-label="Leave request tabs">
      <button
        role="tab"
        class="dashboard-tabs__tab"
        :class="{ active: activeTab === 'form' }"
        :aria-selected="activeTab === 'form'"
        @click="activeTab = 'form'"
      >
        Form
      </button>
      <button
        role="tab"
        class="dashboard-tabs__tab"
        :class="{ active: activeTab === 'requests' }"
        :aria-selected="activeTab === 'requests'"
        @click="activeTab = 'requests'"
      >
        Leave Requests
      </button>
    </nav>

    <div v-if="!isAdmin || activeTab === 'form'" class="leave-form-wrap">
    <form class="card" @submit.prevent="submitLeave">

      <div class="form-grid">

        <FormField
  label="Employee Name"
  v-model="form.employeeName"
  placeholder="Enter your full name"
  :error="errors.employeeName"
  @input="clearErr('employeeName')"
/>

<FormField
  label="Department Manager Email"
  type="text"
  v-model="form.departmentManager"
  placeholder="Enter department manager email"
  :readonly="!isAdmin"
  :error="errors.departmentManager"
  @input="clearErr('departmentManager')"
/>

        <div ref="deptDropdownRef" class="field custom-select">
          <label class="mono-label">Department</label>
          <button
            type="button"
            class="custom-select__trigger"
            :class="{ error: errors.department }"
            @click.stop="deptDropdownOpen = !deptDropdownOpen"
          >
            <span class="custom-select__label">{{
              form.department
                ? departmentOptions.find((d) => d.value === form.department)?.label
                : 'Select Department'
            }}</span>
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
          <span v-if="errors.department" class="err-msg">{{ errors.department }}</span>
          <div v-if="deptDropdownOpen" class="custom-select__options">
            <button
              type="button"
              class="custom-select__option"
              :class="{ selected: !form.department }"
              @click="selectDepartment('')"
            >
              Select Department
            </button>
            <button
              v-for="dept in departmentOptions"
              :key="dept.value"
              type="button"
              class="custom-select__option"
              :class="{ selected: form.department === dept.value }"
              @click="selectDepartment(dept.value)"
            >
              {{ dept.label }}
            </button>
          </div>
        </div>

        <div ref="leaveDropdownRef" class="field custom-select">
          <label class="mono-label">Leave Type</label>
          <button
            type="button"
            class="custom-select__trigger"
            :class="{ error: errors.leaveType }"
            @click.stop="leaveDropdownOpen = !leaveDropdownOpen"
          >
            <span class="custom-select__label">{{
              form.leaveType
                ? leaveOptions.find((l) => l.value === form.leaveType)?.label
                : 'Select leave type'
            }}</span>
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
          <span v-if="errors.leaveType" class="err-msg">{{ errors.leaveType }}</span>
          <div v-if="leaveDropdownOpen" class="custom-select__options">
            <button
              type="button"
              class="custom-select__option"
              :class="{ selected: !form.leaveType }"
              @click="selectLeaveType('')"
            >
              Select leave type
            </button>
            <button
              v-for="leave in leaveOptions"
              :key="leave.value"
              type="button"
              class="custom-select__option"
              :class="{ selected: form.leaveType === leave.value }"
              @click="selectLeaveType(leave.value)"
            >
              {{ leave.label }}
            </button>
          </div>
        </div>

        <FormField
          type="date"
          label="Start Date"
          v-model="form.startDate"
          :error="errors.startDate"
          @input="clearErr('startDate')"
        />

        <FormField
          type="date"
          label="End Date"
          v-model="form.endDate"
          :error="errors.endDate"
          @input="clearErr('endDate')"
        />

        <FormField
          class="full-width"
          type="textarea"
          label="Reason"
          v-model="form.reason"
          placeholder="State the reason for your leave request"
          :error="errors.reason"
          @input="clearErr('reason')"
        />
                <FileUpload
                class="full-width"
                label="Supporting Documents (Optional)"
                v-model:files="attachments"
                @preview="openFilePreview"
                />
      </div>

      <div class="button-group">
  <button
  type="button"
  class="btn btn-primary"
  @click="openPreview"
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

  Review &amp; Submit
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

  <div class="preview-head">
    <div>
      <p class="company-label">Getpayed Technology Solutions Ltd.</p>
      <h1 class="preview-title serif">Leave Request Form</h1>
    </div>
  </div>

  <div class="preview-row">
    <span class="preview-label">Employee Name</span>
    <span class="preview-value">{{ form.employeeName || '—' }}</span>
  </div>

  <div class="preview-row">
    <span class="preview-label">Department Manager</span>
    <span class="preview-value">{{ form.departmentManager || '—' }}</span>
  </div>

  <div class="preview-row">
    <span class="preview-label">Department</span>
    <span class="preview-value">{{ form.department || '—' }}</span>
  </div>

  <div class="preview-row">
    <span class="preview-label">Leave Type</span>
    <span class="preview-value">{{ leaveOptions.find((l) => l.value === form.leaveType)?.label || '—' }}</span>
  </div>

  <div class="preview-row">
    <span class="preview-label">Start Date</span>
    <span class="preview-value">{{ form.startDate || '—' }}</span>
  </div>

  <div class="preview-row">
    <span class="preview-label">End Date</span>
    <span class="preview-value">{{ form.endDate || '—' }}</span>
  </div>

  <div class="preview-row preview-row--block">
    <span class="preview-label">Reason</span>
    <span class="preview-value">{{ form.reason || '—' }}</span>
  </div>

  <div class="preview-row">
    <span class="preview-label">Attachments</span>
    <span class="preview-value">
      <template v-if="attachments.length">
        <span v-for="(file, i) in attachments" :key="i">
          <a
            href="#"
            class="file-link"
            @click.prevent="openFilePreview(file)"
          >
            {{ file.name }}
          </a>{{ i < attachments.length - 1 ? ', ' : '' }}
        </span>
      </template>
      <template v-else>None</template>
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
 
        <FilePreview
        :show="showFilePreview"
        :file="previewFile"
        @close="showFilePreview = false"
        />
    </div>

    <div v-if="isAdmin && activeTab === 'requests'" class="card requests-card">
      <h2 class="serif">Leave Requests</h2>
      <p class="vouchers-sub">{{ displayedLeaveRequests.length }} leave request{{ displayedLeaveRequests.length !== 1 ? 's' : '' }} submitted by your team</p>

      <VoucherTableSkeleton v-if="loadingLeaveRequests" />

      <div v-else-if="!displayedLeaveRequests.length" class="vouchers-empty card">
        <p class="vouchers-empty__title">No leave requests</p>
        <p class="vouchers-empty__sub">Leave requests from users you onboarded will appear here.</p>
      </div>

      <div v-else class="vouchers-table-wrap card">
        <table class="vouchers-table">
          <thead>
            <tr>
              <th>Date</th>
              <th>Employee</th>
              <th>Department</th>
              <th>Leave Type</th>
              <th>Start</th>
              <th>End</th>
              <th class="text-center">Status</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="leave in displayedLeaveRequests"
              :key="leave.id"
              class="vouchers-table__row"
            >
              <td class="text-muted">{{ new Date(leave.createdAt).toLocaleDateString() }}</td>
              <td class="font-medium">{{ leave.employeeName }}</td>
              <td class="text-muted">{{ leave.department }}</td>
              <td>{{ leave.leaveType }}</td>
              <td class="text-muted">{{ leave.startDate }}</td>
              <td class="text-muted">{{ leave.endDate }}</td>
              <td class="text-center">
                <span class="status-badge" :class="'status-badge--' + (leave.status?.toLowerCase() || 'pending')">
                  {{ leave.status || 'Pending' }}
                </span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, computed, onMounted, onBeforeUnmount, watch } from 'vue'
import {
  isAdmin,
  userCreatedBy,
  userEmail,
  userRole,
  fetchCurrentUser,
  fetchOnboardingUsers,
  fetchLeaveRequests,
  addLeaveRequest,
  onboardDepts,
  onboardingUsers,
  allLeaveRequests,
  loadingLeaveRequests,
} from '~/composables/appState'
import VoucherTableSkeleton from '../components/VoucherTableSkeleton.vue'

const showPreview = ref(false)
const showFilePreview = ref(false)
const previewFile = ref(null)
const submitting = ref(false)
const form = reactive({
  employeeName: '',
  departmentManager: userCreatedBy.value || '',
  department: '',
  leaveType: '',
  startDate: '',
  endDate: '',
  reason: '',
})
const attachments = ref([])
const errors = reactive({})

const activeTab = ref('form')
const isSuperAdmin = computed(() => userRole.value === 'super admin')
const onboardingUserEmails = computed(() => onboardingUsers.value.map((u) => u.email))
const displayedLeaveRequests = computed(() => {
  if (isSuperAdmin.value) return allLeaveRequests.value
  return allLeaveRequests.value.filter((leave) =>
    onboardingUserEmails.value.includes(leave.submittedBy),
  )
})

function validate() {
  errors.employeeName = form.employeeName.trim() ? '' : 'Employee name is required'
  const manager = form.departmentManager.trim()
  const isValidEmail = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(manager)
  errors.departmentManager = manager
    ? (isValidEmail ? '' : 'Enter a valid email')
    : 'Department manager is required'
  errors.department = form.department ? '' : 'Department is required'
  errors.leaveType = form.leaveType ? '' : 'Leave type is required'
  errors.startDate = form.startDate ? '' : 'Start date is required'
  errors.endDate = form.endDate ? '' : 'End date is required'
  errors.reason = form.reason.trim() ? '' : 'Reason is required'
  return Object.values(errors).every((e) => !e)
}

function openPreview() {
  if (validate()) showPreview.value = true
}

function clearErr(key) {
  delete errors[key]
}

const leaveOptions = [
  { label: 'Annual Leave', value: 'annual' },
  { label: 'Sick Leave', value: 'sick' },
  { label: 'Casual Leave', value: 'casual' },
  { label: 'Maternity Leave', value: 'maternity' },
  { label: 'Paternity Leave', value: 'paternity' },
]

const departmentOptions = computed(() =>
  onboardDepts.value.map((d) => ({ label: d, value: d }))
)

const deptDropdownOpen = ref(false)
const leaveDropdownOpen = ref(false)
const deptDropdownRef = ref(null)
const leaveDropdownRef = ref(null)

function selectDepartment(value) {
  form.department = value
  deptDropdownOpen.value = false
  clearErr('department')
}

function selectLeaveType(value) {
  form.leaveType = value
  leaveDropdownOpen.value = false
  clearErr('leaveType')
}

function closeDropdowns(event) {
  if (deptDropdownRef.value && !deptDropdownRef.value.contains(event.target)) {
    deptDropdownOpen.value = false
  }
  if (leaveDropdownRef.value && !leaveDropdownRef.value.contains(event.target)) {
    leaveDropdownOpen.value = false
  }
}

onMounted(async () => {
  try {
    await fetchCurrentUser()
  } catch {
    // keep localStorage values
  }
  try {
    if (isAdmin.value) {
      await fetchOnboardingUsers()
      await fetchLeaveRequests()
    }
  } catch {
    // ignore
  }
  form.departmentManager = userCreatedBy.value || ''
  window.addEventListener('click', closeDropdowns)

  const saved = localStorage.getItem('pcv_leave_tab')
  if (saved === 'form' || saved === 'requests') activeTab.value = saved
})
onBeforeUnmount(() => window.removeEventListener('click', closeDropdowns))

watch(activeTab, (value) => {
  localStorage.setItem('pcv_leave_tab', value)
})

function openFilePreview(file) {
  previewFile.value = file
  showFilePreview.value = true
}

function resetForm() {
  Object.assign(form, {
    employeeName: '',
    departmentManager: userCreatedBy.value || '',
    department: '',
    leaveType: '',
    startDate: '',
    endDate: '',
    reason: '',
  })
  attachments.value = []
  Object.keys(errors).forEach((key) => delete errors[key])
}

async function submitLeave() {
  if (!validate()) return
  if (submitting.value) return
  submitting.value = true
  try {
    await addLeaveRequest({
      ...form,
      attachments: attachments.value,
      submittedBy: userEmail.value,
    })
    showPreview.value = false
    resetForm()
    if (isAdmin.value) activeTab.value = 'requests'
  } catch (err) {
    console.error(err)
    alert(err.message || 'Failed to submit leave request')
  } finally {
    submitting.value = false
  }
}
</script>

<style scoped>
.page {
  max-width: 1000px;
  margin: 0 auto;
  padding: 40px 0;
}

.page-header {
  width: 750px;
  margin: 16px auto 40px;
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
  max-width: 1000px;
  margin: 0 auto;
  padding-top: 24px;
}

.page-header {
  max-width: 900px;
  margin-left: auto;
  margin-right: auto;
}


.dashboard-tabs {
  width: 800px !important;
  margin-top: 24px;
  margin-left: auto;
  margin-right: auto;
}

.dashboard-tabs .dashboard-tabs__tab {
  flex: 1;
  justify-content: center;
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


/* ── Preview header ─────────────────────────────────────────── */
.preview-title {
  font-family: var(--font-serif);
  font-size: 20px;
  font-weight: 700;
  margin-top: 4px;
}

.preview-row {
  display: flex;
  align-items: center;
  gap: 20px;
  padding: 4px 0;
  font-size: 14px;
  line-height: 1.6;
}

.preview-row > span:first-child {
  flex-shrink: 0;
  width: 160px;
  font-family: var(--font-mono);
  font-size: 10.5px;
  color: var(--muted-fg);
  text-transform: uppercase;
  letter-spacing: 0.12em;
}

.preview-row > span:last-child {
  flex: 1;
  color: var(--fg);
  word-break: break-word;
}

.preview-row.preview-row--block {
  flex-direction: column;
  gap: 8px;
  align-items: stretch;
}

.preview-row.preview-row--block > span:first-child {
  width: auto;
  align-self: flex-start;
}

.button-group {
  display: flex;
  justify-content: flex-end;
  margin-top: 44px;
}

.review-btn {
  width: 100%;
  justify-content: center;
  padding: 14px;
  font-size: 15px;
  margin-top: 4px;
}
.custom-select__trigger.error {
  border-color: color-mix(in srgb, var(--destructive) 60%, transparent);
}
.modal-footer {
  padding-top: 2px;
}

.modal-body {
  padding: 32px;
}

.preview-head {
  justify-content: flex-start;
}

.preview-row {
  justify-content: flex-start;
}

.requests-card h2 {
  font-size: 20px;
  margin: 0 0 4px;
}

.requests-card .vouchers-sub {
  margin-bottom: 24px;
}
</style>