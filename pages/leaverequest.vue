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
            {{ activeTab === 'requests' ? (isAdmin ? 'Leave Requests' : 'My Leave Requests') : 'Leave Request Form' }}
          </h1>

          <p v-if="activeTab === 'requests' && isAdmin" class="vouchers-sub">
            {{ displayedLeaveRequests.length }} leave request{{ displayedLeaveRequests.length !== 1 ? 's' : '' }} submitted by users
          </p>
        </div>

      </div>

    </div>

    <nav class="dashboard-tabs" role="tablist" aria-label="Leave request tabs">
      <button
        role="tab"
        class="dashboard-tabs__tab"
        :class="{ active: activeTab === 'form' }"
        :aria-selected="activeTab === 'form'"
        @click="setLeaveTab('form')"
      >
        Form
      </button>
      <button
        role="tab"
        class="dashboard-tabs__tab"
        :class="{ active: activeTab === 'requests' }"
        :aria-selected="activeTab === 'requests'"
        @click="setLeaveTab('requests')"
      >
        Leave Requests
      </button>
    </nav>

    <div v-if="activeTab === 'form'" class="leave-form-wrap">
      <div v-if="submitted" class="card success-card">
        <div class="success-icon">✓</div>
        <h2 class="serif">Leave Request Submitted</h2>
        <p>Your leave request has been sent for review.</p>
        <code class="leave-badge">{{ submittedEmployee }}</code>
        <div class="success-actions">
          <button v-if="isAdmin" class="btn btn-outline" @click="viewRequests">View Leave Requests</button>
          <button class="btn btn-primary" @click="resetForm">New Leave Request</button>
        </div>
      </div>
      <form v-if="!submitted" class="card" @submit.prevent="submitLeave">

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

        <FormField
  label="Department"
  v-model="form.department"
  :readonly="true"
  :error="errors.department"
  @input="clearErr('department')"
/>

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

    <!-- Modal Body/Form Preview -->

    <div class="modal-body">

  <div class="preview-head">
    <div>
      <p class="company-label">Getpayed Technology Solutions Ltd.</p>
      <h1 class="preview-title serif">Leave Request Form</h1>
    </div>
  </div>

  <div class="preview-row">
    <span class="preview-label">Employee Name</span>
    <span class="preview-value">{{ form.employeeName }}</span>
  </div>

  <div class="preview-row">
    <span class="preview-label">Department Manager</span>
    <span class="preview-value">{{ form.departmentManager }}</span>
  </div>

  <div class="preview-row">
    <span class="preview-label">Department</span>
    <span class="preview-value">{{ form.department }}</span>
  </div>

  <div class="preview-row">
    <span class="preview-label">Leave Type</span>
    <span class="preview-value">{{ leaveOptions.find((l) => l.value === form.leaveType)?.label || '—' }}</span>
  </div>

  <div class="preview-row">
    <span class="preview-label">Start Date</span>
    <span class="preview-value">{{ form.startDate }}</span>
  </div>

  <div class="preview-row">
    <span class="preview-label">End Date</span>
    <span class="preview-value">{{ form.endDate }}</span>
  </div>

  <div class="preview-row preview-row--block">
    <span class="preview-label">Reason</span>
    <span class="preview-value">{{ form.reason }}</span>
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
        :disabled="submitting"
        class="btn btn-primary"
        @click="submitLeave"
      >
        {{ submitting ? 'Submitting...' : 'Submit Form' }}
      </button>
      </div>

    </div>
  </div>
 
    </div>

    <!-- Requests tab -->
    <div v-show="activeTab === 'requests'">
      <div v-if="isAdmin" class="admin-filters card">
        <div class="filter-row">
          <input
            v-model="filterText"
            type="text"
            class="filter-input"
            placeholder="Filter by employee, status"
          />
          <button class="btn btn-primary" @click="clearFilter">Clear Filters</button>
        </div>
      </div>

      <VoucherTableSkeleton v-if="loadingLeaveRequests" :columns="7" />

      <template v-else>
      <div class="vouchers-table-wrap card">

        <div v-if="!displayedLeaveRequests.length" class="vouchers-empty">
          <p class="vouchers-empty__title">No leave requests</p>
          <p class="vouchers-empty__sub">{{ isAdmin ? 'Leave requests from users you onboarded will appear here.' : 'Your leave requests will appear here.' }}</p>
        </div>

        <table v-else class="vouchers-table">
          <thead>
            <tr>
              <th>Date</th>
              <th>Employee</th>
              <th>Leave Period</th>
              <th>Leave Type</th>
              <th class="reason-col">Reason</th>
              <th>Attachments</th>
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
              <td class="text-muted">{{ formatLeavePeriod(leave) }}</td>
              <td>{{ leave.leaveType }}</td>
              <td class="reason-cell">
                <span
                  class="reason-text"
                  :class="{ expanded: expandedReasons[leave.id] }"
                  @click.stop="toggleReason(leave.id)"
                >
                  {{ leave.reason }}
                </span>
              </td>
              <td class="attachments-cell">
                <template v-if="leave.attachments && leave.attachments.length">
                  <span v-for="(file, i) in leave.attachments" :key="i">
                    <a
                      href="#"
                      class="file-link"
                      @click.prevent="openFilePreview(file)"
                    >
                      {{ typeof file === 'string' ? file : (file.name || '-') }}
                    </a>{{ i < leave.attachments.length - 1 ? ', ' : '' }}
                  </span>
                </template>
                <template v-else>—</template>
              </td>
              <td class="text-center">
                <span class="status-wrap">
                  <span class="status-badge" :class="'status-badge--' + (leave.status?.toLowerCase() || 'pending')">
                    {{ leave.status || 'Pending' }}
                  </span>
                  <button
                    v-if="isAdmin && (leave.status || 'Pending').toLowerCase() === 'pending'"
                    type="button"
                    class="info-btn"
                    title="Click to approve/decline"
                    aria-label="Approve or decline"
                    @click.stop="openActionModal(leave)"
                  >
                    <span class="info-icon">i</span>
                  </button>
                </span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      </template>
    </div>

    <!-- Approve/Decline modal -->
  <div v-if="showActionModal" class="modal-backdrop" @click.self="showActionModal = false">
    <div class="modal" role="dialog" aria-modal="true" aria-label="Approve or decline leave" style="max-width: 640px;">
      <div class="modal-header" style="padding: 8px 24px 4px;">
        <div class="modal-header__title" style="font-size: 16px; font-weight: 700; letter-spacing: -0.04em;">Approve or decline?</div>
        <button class="modal-close" @click="showActionModal = false" aria-label="Close">✕</button>
      </div>
      <div class="modal-body" style="padding: 12px 24px; gap: 0;">
        <p style="font-size: 16px; color: var(--muted-fg); margin: 6px 0 0;">Choose an action for this leave request.</p>
        <div style="display: flex; justify-content: flex-end; gap: 12px; margin-top: 12px; transform: translateY(-4px);">
          <button class="btn btn-approve" style="border-radius: 9999px; padding: 9px 28px; font-size: 15px;" @click="chooseAction('Approved')">Approve</button>
          <button class="btn" :class="isSuperAdmin ? 'btn-decline-subtle' : 'btn-decline'" style="border-radius: 9999px; padding: 9px 28px; font-size: 15px;" @click="chooseAction('Declined')">Decline</button>
        </div>
      </div>
    </div>
  </div>

  <div v-if="showConfirmModal" class="modal-backdrop" @click.self="showConfirmModal = false">
    <div class="modal" role="dialog" aria-modal="true" :aria-label="(pendingAction === 'Approved' ? 'Approve' : 'Decline') + ' confirmation'" style="max-width: 640px;">
      <div class="modal-header" style="padding: 8px 24px 4px;">
        <div class="modal-header__title" style="font-size: 16px; font-weight: 700; letter-spacing: -0.04em;">{{ pendingAction === 'Approved' ? 'Approve leave?' : 'Decline leave?' }}</div>
        <button class="modal-close" @click="showConfirmModal = false" aria-label="Close">✕</button>
      </div>
      <div class="modal-body">
        <p style="font-size: 18px; font-weight: 900; letter-spacing: -0.04em; margin: 0;">
          Are you sure you want to {{ pendingAction === 'Approved' ? 'approve' : 'decline' }} this leave request?
          <span style="display: block; margin-top: 12px; font-size: 14px; color: var(--muted-fg); font-weight: 500;">This action cannot be undone.</span>
        </p>
      </div>
      <div class="modal-footer" style="border-top: none;">
        <button class="btn btn-outline" style="border-radius: 9999px;" @click="cancelConfirm">Cancel</button>
        <button class="btn" :class="pendingAction === 'Approved' ? 'btn-approve' : 'btn-decline'" :disabled="processing" style="border-radius: 9999px;" @click="confirmStatus">
          {{ processing && processingAction === pendingAction ? (pendingAction === 'Approved' ? 'Approving…' : 'Declining…') : (pendingAction === 'Approved' ? 'Yes, Approve' : 'Yes, Decline') }}
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
</template>

<script setup>
import { reactive, ref, computed, onMounted, onBeforeUnmount } from 'vue'
import { useCookie } from '#app'
import {
  isAdmin,
  userCreatedBy,
  userEmail,
  userRole,
  userDepartment,
  fetchCurrentUser,
  fetchLeaveRequests,
  fetchOnboardingUsers,
  addLeaveRequest,
  updateLeaveRequestStatus,
  onboardDepts,
  allLeaveRequests,
  loadingLeaveRequests,
  onboardingUsers,
} from '~/composables/appState'
import VoucherTableSkeleton from '../components/VoucherTableSkeleton.vue'

const showPreview = ref(false)
const showFilePreview = ref(false)
const previewFile = ref(null)
const submitting = ref(false)
const submitted = ref(false)
const submittedEmployee = ref('')
const form = reactive({
  employeeName: '',
  departmentManager: userCreatedBy.value || '',
  department: userRole.value === 'super admin' ? 'Finance' : (userDepartment.value || ''),
  leaveType: '',
  startDate: '',
  endDate: '',
  reason: '',
})
const attachments = ref([])
const errors = reactive({})

const activeTab = useCookie('pcv_leave_tab', { default: () => 'form' })
const expandedReasons = reactive({})
const filterText = ref('')
const selectedLeave = ref(null)
const showActionModal = ref(false)
const showConfirmModal = ref(false)
const pendingAction = ref('')
const processing = ref(false)
const processingAction = ref('')
const isSuperAdmin = computed(() => userRole.value === 'super admin')

function setLeaveTab(tab) {
  activeTab.value = tab
}

function toggleReason(id) {
  expandedReasons[id] = !expandedReasons[id]
}

function clearFilter() {
  filterText.value = ''
}

function openActionModal(leave) {
  selectedLeave.value = leave
  showActionModal.value = true
}

function chooseAction(action) {
  pendingAction.value = action
  showActionModal.value = false
  showConfirmModal.value = true
}

function cancelConfirm() {
  showConfirmModal.value = false
  showActionModal.value = true
}

async function confirmStatus() {
  if (processing.value || !selectedLeave.value || !pendingAction.value) return
  processing.value = true
  processingAction.value = pendingAction.value
  try {
    await updateLeaveRequestStatus(selectedLeave.value.id, pendingAction.value)
  } catch (err) {
    console.error(err)
    alert(err.message || 'Failed to update leave request status')
  } finally {
    processing.value = false
    processingAction.value = ''
  }
  showConfirmModal.value = false
  showActionModal.value = false
  selectedLeave.value = null
}

const displayedLeaveRequests = computed(() => {
  const myEmail = String(userEmail.value || '').toLowerCase()
  const canViewAll = myEmail === 'hr@getpayedmail.com'
  const onboardedEmails = new Set(onboardingUsers.value.map((user) => String(user.email || '').toLowerCase()))
  let requests
  if (canViewAll) {
    requests = allLeaveRequests.value
  } else if (isAdmin.value) {
    requests = allLeaveRequests.value.filter((leave) => {
      const submittedBy = String(leave.submittedBy || '').toLowerCase()
      return submittedBy === myEmail || onboardedEmails.has(submittedBy)
    })
  } else {
    requests = allLeaveRequests.value.filter((leave) =>
      String(leave.submittedBy || '').toLowerCase() === myEmail,
    )
  }

  const query = filterText.value.trim().toLowerCase()
  if (query) {
    requests = requests.filter((leave) =>
      (leave.employeeName?.toLowerCase().includes(query) || false) ||
      (leave.status?.toLowerCase().includes(query) || false),
    )
  }

  return requests
})

function parseDate(value) {
  const [year, month, day] = value.split('-').map(Number)
  return new Date(year, month - 1, day)
}

function validate() {
  const today = new Date()
  today.setHours(0, 0, 0, 0)

  errors.employeeName = form.employeeName.trim() ? '' : 'Employee name is required'
  const manager = form.departmentManager.trim()
  if (!manager) {
    errors.departmentManager = 'Department manager is required'
  } else if (isAdmin.value || isSuperAdmin.value) {
    errors.departmentManager = /^[^\s@]+@getpayedmail\.com$/.test(manager.toLowerCase())
      ? ''
      : 'Manager email must be a getpayedmail.com address'
  } else {
    errors.departmentManager = ''
  }
  errors.department = form.department ? '' : 'Department is required'
  errors.leaveType = form.leaveType ? '' : 'Leave type is required'

  if (!form.startDate) {
    errors.startDate = 'Start date is required'
  } else {
    const start = parseDate(form.startDate)
    errors.startDate = start < today ? 'Enter a valid start date' : ''
  }

  if (!form.endDate) {
    errors.endDate = 'End date is required'
  } else {
    const end = parseDate(form.endDate)
    errors.endDate = end <= today ? 'Enter a valid end date' : ''
  }

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
  { label: 'Annual Leave', value: 'Annual' },
  { label: 'Sick Leave', value: 'Sick' },
  { label: 'Casual Leave', value: 'Casual' },
  { label: 'Maternity Leave', value: 'Maternity' },
  { label: 'Paternity Leave', value: 'Paternity' },
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
    if (!allLeaveRequests.value.length) await fetchLeaveRequests()
  } catch {
    // ignore
  }
  if (isAdmin.value) {
    try {
      await fetchOnboardingUsers()
    } catch {
      // ignore
    }
  }
  form.departmentManager = userCreatedBy.value || ''
  form.department = isSuperAdmin.value ? 'Finance' : (userDepartment.value || '')
  window.addEventListener('click', closeDropdowns)
})
onBeforeUnmount(() => window.removeEventListener('click', closeDropdowns))


function formatDate(value) {
  if (!value) return ''
  return new Date(value).toLocaleDateString('en-US', { month: 'long', day: 'numeric', year: 'numeric' })
}

function formatLeavePeriod(leave) {
  if (!leave.startDate || !leave.endDate) return '—'
  return `${formatDate(leave.startDate)} - ${formatDate(leave.endDate)}`
}

function formatAttachments(attachments) {
  if (!attachments || !attachments.length) return '—'
  return attachments.map((file) => (typeof file === 'string' ? file : file.name) || '—').join(', ')
}

function openFilePreview(file) {
  previewFile.value = typeof file === 'string' ? { name: file, data: file } : file
  showFilePreview.value = true
}

function resetForm() {
  Object.assign(form, {
    employeeName: '',
    departmentManager: userCreatedBy.value || '',
    department: isSuperAdmin.value ? 'Finance' : (userDepartment.value || ''),
    leaveType: '',
    startDate: '',
    endDate: '',
    reason: '',
  })
  attachments.value = []
  Object.keys(errors).forEach((key) => delete errors[key])
  submitted.value = false
  submittedEmployee.value = ''
}

function viewRequests() {
  activeTab.value = 'requests'
  resetForm()
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
    submittedEmployee.value = form.employeeName
    submitted.value = true
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
  width: 100%;
  max-width: 750px;
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
  padding-top: 24px;
}

.dashboard-tabs {
  width: 100%;
  max-width: 800px;
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
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 28px;
}

.full-width {
  grid-column: 1 / -1;
}

.btn {
  margin-top: 28px;
}


.leave-form-wrap {
  padding: 0 16px;
}

@media (max-width: 768px) {

  .page {
    padding: 16px;
  }

  .page-header {
    margin: 8px auto 24px;
  }

  .page-header__left {
    gap: 10px;
  }

  .header-icon {
    width: 38px;
    height: 38px;
    margin-top: 2px;
  }

  .serif {
    font-size: 18px;
  }

  .form-grid {
    grid-template-columns: minmax(0, 1fr);
    gap: 16px;
  }

  .card {
    padding: 20px 16px;
  }

  .content {
    padding-top: 12px;
  }

  .button-group {
    margin-top: 24px;
    justify-content: stretch;
  }

  .button-group .btn {
    width: 100%;
    justify-content: center;
  }

  .content .vouchers-table-wrap {
    overflow-x: auto;
    -webkit-overflow-scrolling: touch;
  }

  .content .vouchers-table {
    min-width: 600px;
  }

  .filter-row {
    flex-wrap: wrap;
  }

  .filter-row .btn {
    transform: none;
  }

  .admin-filters {
    padding: 14px 16px;
  }

  .preview-row {
    align-items: flex-start;
  }

  .preview-row > span:first-child {
    width: 120px;
  }

  .modal-body {
    padding: 16px;
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

.content .vouchers-table-wrap {
  max-width: 1040px;
  padding: 0;
  margin-bottom: 20px;
  margin-left: auto;
  margin-right: auto;
  overflow-y: hidden;
  border-radius: 0;
}

.content .vouchers-empty {
  max-width: 1040px;
  padding: 60px 32px;
  margin-bottom: 0;
  margin-left: auto;
  margin-right: auto;
  border-radius: 0;
}

.content .vouchers-table th,
.content .vouchers-table td {
  border-right: 1px solid var(--border);
}

.content .vouchers-table th:last-child,
.content .vouchers-table td:last-child {
  border-right: none;
}

.content .vouchers-table .reason-col,
.content .vouchers-table .reason-cell {
  width: 30%;
  min-width: 250px;
}

.content .vouchers-table .reason-text {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  line-clamp: 2;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: normal;
  overflow-wrap: anywhere;
  cursor: pointer;
  color: #000;
}

.content .vouchers-table .reason-text.expanded {
  -webkit-line-clamp: unset;
  line-clamp: unset;
  display: block;
}

.content .vouchers-table .attachments-cell {
  white-space: normal;
  overflow-wrap: break-word;
}

.content .vouchers-table__row:hover td {
  background: transparent;
}

.filter-row {
  display: flex;
  flex-wrap: nowrap;
  align-items: center;
  gap: 12px;
  padding: 10px 16px;
  border-bottom: 1px solid var(--border);
}

.filter-input {
  flex: 1 1 0%;
  min-width: 0;
  height: 38px;
  padding: 0 12px;
  border: 1px solid var(--border);
  border-radius: 8px;
  font-size: 14px;
}

.filter-input:focus {
  background: var(--input-bg);
  border-color: color-mix(in srgb, var(--primary) 50%, transparent);
  box-shadow: 0 0 0 3px color-mix(in srgb, var(--primary) 12%, transparent);
}

.filter-row .btn {
  flex: 0 0 auto;
  height: 34px;
  padding: 0 16px;
  white-space: nowrap;
  font-size: 13px;
  display: inline-flex;
  align-items: center;
  transform: translateY(-14px);
}
.admin-filters {
  padding: 18px 20px;
  margin: 0 auto 16px;
  max-width: 1040px;
}

.status-wrap {
  display: inline-flex;
  align-items: center;
  gap: 4px;
}

.info-btn {
  width: 13px;
  height: 13px;
  border-radius: 50%;
  border: 2px solid var(--border);
  background: transparent;
  color: var(--muted-fg);
  font-size: 9px;
  font-weight: 800;
  line-height: 1;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 0;
  transition: background 0.15s, color 0.15s;
}

.info-btn:hover {
  background: var(--muted-fg);
  border-color: var(--muted-fg);
  color: var(--primary-fg);
}

.info-icon {
  display: inline-block;
}

.admin-filters .filter-row {
  border-bottom: none;
  padding: 0;
  flex: 1 1 100%;
}

.leave-badge {
  font-family: var(--font-mono);
  font-size: 14px;
  color: var(--accent);
  display: block;
  margin: 8px 0 24px;
}

@media (max-width: 480px) {
  .page {
    padding: 12px;
  }

  .serif {
    font-size: 16px;
  }

  .card {
    padding: 16px 12px;
  }

  .header-icon {
    width: 34px;
    height: 34px;
  }

  .button-group {
    margin-top: 20px;
  }

  .content .vouchers-table {
    min-width: 500px;
  }

  .content .vouchers-table th,
  .content .vouchers-table td {
    padding: 8px 10px;
    font-size: 12px;
  }

  .content .vouchers-table .reason-col,
  .content .vouchers-table .reason-cell {
    min-width: 150px;
  }

  .filter-input {
    font-size: 13px;
  }

  .modal-body {
    padding: 12px;
  }

  .preview-row {
    gap: 4px;
  }
}
</style>