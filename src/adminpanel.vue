<template>
  <div class="page-wrap">
    <div class="vouchers-header">
      <div>
        <h1 class="serif vouchers-title">Dashboard</h1>
        <p class="vouchers-sub">
          {{
            activeTab === 'vouchers'
              ? filteredAdminVouchers.length + ' submitted voucher' + (filteredAdminVouchers.length !== 1 ? 's' : '') + ' across all users'
              : 'Manage user access and onboarding'
          }}
        </p>
      </div>
      <div class="admin-header-actions">
        <span class="admin-badge mono-label">
          <svg
            width="12"
            height="12"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
          >
            <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2" />
            <circle cx="9" cy="7" r="4" />
            <path d="M23 21v-2a4 4 0 0 0-3-3.87" />
            <path d="M16 3.13a4 4 0 0 1 0 7.75" />
          </svg>
          {{ userRole === 'super admin' ? 'SUPER ADMIN VIEW' : 'ADMIN VIEW' }}
        </span>
      </div>
    </div>

    <nav class="dashboard-tabs" role="tablist" aria-label="Dashboard sections">
      <button
        role="tab"
        class="dashboard-tabs__tab"
        :class="{ active: activeTab === 'vouchers' }"
        :aria-selected="activeTab === 'vouchers'"
        @click="activeTab = 'vouchers'"
      >
        All vouchers
      </button>
      <button
        role="tab"
        class="dashboard-tabs__tab"
        :class="{ active: activeTab === 'onboarding' }"
        :aria-selected="activeTab === 'onboarding'"
        @click="activeTab = 'onboarding'"
      >
        User Onboarding
      </button>
    </nav>

    <!-- All vouchers tab -->
    <div v-if="activeTab === 'vouchers'" role="tabpanel">
      <!-- Voucher detail view -->
      <div v-if="selectedVoucher" class="voucher-detail">
        <button class="btn btn-outline voucher-detail__back" @click="selectedVoucher = null">
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

        <div class="card">
          <div class="preview-head">
            <div>
              <p class="company-label">Getpayed Technology Solutions Ltd.</p>
              <h2 class="preview-title serif">Petty Cash Voucher</h2>
            </div>
            <div class="preview-head__right">
              <p class="mono-label tiny">Voucher No.</p>
              <code class="voucher-no">{{ selectedVoucher.id }}</code>
            </div>
          </div>

          <div class="preview-section">
            <h3 class="preview-section__title mono-label">Email Details</h3>
            <div class="preview-rows">
              <div class="preview-row">
                <span>From</span><span>{{ selectedVoucher.from || '—' }}</span>
              </div>
              <div class="preview-row">
                <span>To</span><span>{{ selectedVoucher.to || '—' }}</span>
              </div>
              <div class="preview-row">
                <span>CC</span><span>{{ selectedVoucher.cc || '—' }}</span>
              </div>
              <div class="preview-row">
                <span>Subject</span><span>{{ selectedVoucher.subject || '—' }}</span>
              </div>
            </div>
          </div>

          <div class="preview-section">
            <h3 class="preview-section__title mono-label">Payee Information</h3>
            <div class="preview-rows">
              <div class="preview-row">
                <span>Payee</span><span>{{ selectedVoucher.payee || '—' }}</span>
              </div>
              <div class="preview-row">
                <span>Department</span><span>{{ selectedVoucher.department || '—' }}</span>
              </div>
            </div>
          </div>

          <div class="preview-section">
            <h3 class="preview-section__title mono-label">Amount &amp; Purpose</h3>
            <div class="preview-rows">
              <div class="preview-row">
                <span>Amount (Figures)</span>
                <span class="mono-input">₦{{ selectedVoucher.amount?.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 }) || '0.00' }}</span>
              </div>
              <div class="preview-row">
                <span>Amount (Words)</span><span>{{ selectedVoucher.amountWords || '—' }}</span>
              </div>
              <div class="preview-row block-row">
                <span>Purpose</span><span>{{ selectedVoucher.purpose || '—' }}</span>
              </div>
            </div>
          </div>

          <div class="preview-section">
            <h3 class="preview-section__title mono-label">Supporting Documents</h3>
            <div class="preview-rows">
              <div class="preview-row">
                <span>Submission Date</span><span>{{ selectedVoucher.submissionDate || '—' }}</span>
              </div>
              <div class="preview-row">
                <span>Attached Files</span>
                <span>
                  <template v-if="selectedVoucher.supportingDocs?.length">
                    <span v-for="(doc, i) in selectedVoucher.supportingDocs" :key="i">
                      <a
                        v-if="typeof doc === 'object' && doc?.data"
                        class="file-link"
                        href="#"
                        @click.prevent="openFilePreview(doc)"
                      >{{ doc.name }}</a>
                      <span v-else>{{ typeof doc === 'string' ? doc : doc?.name || '—' }}</span>
                      {{ i < selectedVoucher.supportingDocs.length - 1 ? ', ' : '' }}
                    </span>
                  </template>
                  <template v-else>None</template>
                </span>
              </div>
            </div>
          </div>

          <div class="amount-banner">
            <span class="mono-label">Total Amount</span>
            <span class="amount-total serif">₦{{ selectedVoucher.amount?.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 }) || '0.00' }}</span>
          </div>
        </div>
      </div>

      <template v-else>
        <div v-if="loadingVouchers" class="vouchers-loading">
          Loading vouchers…
        </div>
        <template v-else>
          <div class="admin-filters card">
          <div v-if="isSuperAdmin" ref="deptDropdownRef" class="field admin-filter-field custom-select">
            <label class="mono-label">Filter by Department</label>
            <button
              type="button"
              class="custom-select__trigger"
              @click.stop="deptDropdownOpen = !deptDropdownOpen"
            >
              <span class="custom-select__label">{{
                adminFilter.dept || 'All Departments'
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
            <div v-if="deptDropdownOpen" class="custom-select__options">
              <button
                type="button"
                class="custom-select__option"
                :class="{ selected: adminFilter.dept === '' }"
                @click="selectDept('')"
              >
                All Departments
              </button>
              <button
                v-for="department in adminDepts"
                :key="department"
                type="button"
                class="custom-select__option"
                :class="{ selected: adminFilter.dept === department }"
                @click="selectDept(department)"
              >
                {{ department }}
              </button>
            </div>
          </div>
          <div ref="userDropdownRef" class="field admin-filter-field custom-select">
            <label class="mono-label">Filter by Submitted By</label>
            <button
              type="button"
              class="custom-select__trigger"
              @click.stop="userDropdownOpen = !userDropdownOpen"
            >
              <span class="custom-select__label">{{
                adminFilter.user || 'All Users'
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
            <div v-if="userDropdownOpen" class="custom-select__options">
              <button
                type="button"
                class="custom-select__option"
                :class="{ selected: adminFilter.user === '' }"
                @click="selectUser('')"
              >
                All Users
              </button>
              <button
                v-for="user in adminUsers"
                :key="user"
                type="button"
                class="custom-select__option"
                :class="{ selected: adminFilter.user === user }"
                @click="selectUser(user)"
              >
                {{ user }}
              </button>
            </div>
          </div>
          <div ref="statusDropdownRef" class="field admin-filter-field custom-select">
            <label class="mono-label">Filter by Status</label>
            <button
              type="button"
              class="custom-select__trigger"
              @click.stop="statusDropdownOpen = !statusDropdownOpen"
            >
              <span class="custom-select__label">{{ formatStatus(adminFilter.status) }}</span>
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
            <div v-if="statusDropdownOpen" class="custom-select__options">
              <button
                type="button"
                class="custom-select__option"
                :class="{ selected: adminFilter.status === '' }"
                @click="selectStatus('')"
              >
                All Statuses
              </button>
              <button
                v-for="status in adminStatuses"
                :key="status"
                type="button"
                class="custom-select__option"
                :class="{ selected: adminFilter.status === status }"
                @click="selectStatus(status)"
              >
                {{ formatStatus(status) }}
              </button>
            </div>
          </div>
          <button
            class="btn btn-outline admin-filter-clear"
            @click="((adminFilter.dept = ''), (adminFilter.user = ''), (adminFilter.status = ''))"
          >
            Clear Filters
          </button>
        </div>

        <div v-if="!filteredAdminVouchers.length" class="vouchers-empty card">
          <svg
            width="40"
            height="40"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="1.5"
          >
            <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2" />
            <circle cx="9" cy="7" r="4" />
            <path d="M23 21v-2a4 4 0 0 0-3-3.87" />
            <path d="M16 3.13a4 4 0 0 1 0 7.75" />
          </svg>
          <p class="vouchers-empty__title">No vouchers found</p>
          <p class="vouchers-empty__sub">No vouchers match the selected filters.</p>
        </div>

        <div v-else class="vouchers-table-wrap card admin-all-vouchers">
          <table class="vouchers-table">
            <thead>
              <tr>
                <th>Voucher No.</th>
                <th>Submitted By</th>
                <th>Date</th>
                <th>Payee</th>
                <th>Department</th>
                <th>Purpose</th>
                <th class="text-right">Amount</th>
                <th class="text-center">Status</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="voucher in filteredAdminVouchers"
                :key="voucher.id + voucher.submittedBy"
                class="vouchers-table__row"
                @click="selectedVoucher = voucher"
              >
                <td>
                  <code class="voucher-cell-no">{{ voucher.id }}</code>
                </td>
                <td>
                  <div class="admin-user-cell">
                    <span class="admin-user-avatar">{{
                      voucher.submittedBy.charAt(0).toUpperCase()
                    }}</span>
                    <span class="admin-user-email">{{ voucher.submittedBy }}</span>
                  </div>
                </td>
                <td class="text-muted">{{ voucher.submissionDate }}</td>
                <td class="font-medium">{{ voucher.payee }}</td>
                <td class="text-muted">{{ voucher.department }}</td>
                <td><span class="voucher-purpose" :class="{ expanded: expandedPurposes[voucher.id] }" @click.stop="togglePurpose(voucher.id)">{{ voucher.purpose }}</span></td>
                <td class="text-right font-mono font-medium">₦{{ voucher.amount.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</td>
                <td class="text-center">
                  <span class="status-badge" :class="'status-badge--' + (voucher.status?.toLowerCase() || 'pending')">
                    <svg v-if="voucher.status === 'Processed'" width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" style="margin-right: 4px;"><polyline points="20 6 9 17 4 12" /></svg>
                    <svg v-if="voucher.status === 'Rejected'" width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" style="margin-right: 4px;"><line x1="18" y1="6" x2="6" y2="18" /><line x1="6" y1="6" x2="18" y2="18" /></svg>
                    {{ voucher.status || 'Pending' }}
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

     
        </template>
      </template>
    </div>

    <!-- User Onboarding tab -->
    <div v-else role="tabpanel">
      <div v-if="isSuperAdmin" class="onboarding-actions" style="display: flex; justify-content: flex-end; margin-bottom: 12px;">
        <button type="button" class="btn btn-accent" @click="openCreateDeptModal" style="gap: 6px;">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round">
            <line x1="12" y1="5" x2="12" y2="19" />
            <line x1="5" y1="12" x2="19" y2="12" />
          </svg>
          Create department
        </button>
      </div>
      <div class="onboarding-form card">
        <h2 class="onboarding-form__title serif">Onboard New User</h2>
        <p class="onboarding-form__sub">Add a user to grant access to the voucher system.</p>
        <form class="onboarding-form__fields" @submit.prevent="handleAddUser">
          <div class="field onboarding-field">
            <label class="mono-label">Email Address </label>
            <input
              v-model="onboardForm.email"
              type="email"
              placeholder="user@getpayedmail.com"
              :class="{ error: onboardErrors.email }"
              @input="delete onboardErrors.email"
            />
            <span v-if="onboardErrors.email" class="err-msg">{{ onboardErrors.email }}</span>
          </div>
          <div v-if="isSuperAdmin" ref="onboardDeptDropdownRef" class="field onboarding-field custom-select">
            <label class="mono-label">Department </label>
            <button
              type="button"
              class="custom-select__trigger"
              :class="{ error: onboardErrors.department }"
              @click.stop="onboardDeptDropdownOpen = !onboardDeptDropdownOpen"
            >
              <span class="custom-select__label">{{ onboardForm.department || 'Select department' }}</span>
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
            <div v-if="onboardDeptDropdownOpen" class="custom-select__options">
              <button
                type="button"
                class="custom-select__option"
                :class="{ selected: !onboardForm.department }"
                @click="selectOnboardDept('')"
              >
                Select department
              </button>
              <div
                v-for="department in onboardDepts"
                :key="department"
                class="custom-select__option custom-select__option--row"
                :class="{ selected: onboardForm.department === department }"
              >
                <button
                  type="button"
                  class="custom-select__option-label"
                  @click="selectOnboardDept(department)"
                >
                  {{ department }}
                </button>
                <div class="custom-select__option-actions">
                  <button
                    type="button"
                    class="icon-btn"
                    @click.stop="openEditDeptModal(department)"
                    aria-label="Edit department"
                  >
                    <svg
                      width="14"
                      height="14"
                      viewBox="0 0 24 24"
                      fill="none"
                      stroke="currentColor"
                      stroke-width="2"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                    >
                      <path d="M17 3a2.828 2.828 0 1 1 4 4L7.5 20.5 2 22l1.5-5.5L17 3z" />
                    </svg>
                  </button>
                  <button
                    type="button"
                    class="icon-btn icon-btn--destructive"
                    @click.stop="openDeleteDeptModal(department)"
                    aria-label="Delete department"
                  >
                    <svg
                      width="14"
                      height="14"
                      viewBox="0 0 24 24"
                      fill="none"
                      stroke="currentColor"
                      stroke-width="2"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                    >
                      <polyline points="3 6 5 6 21 6" />
                      <path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2" />
                      <line x1="10" y1="11" x2="10" y2="17" />
                      <line x1="14" y1="11" x2="14" y2="17" />
                    </svg>
                  </button>
                </div>
              </div>
            </div>
            <span v-if="onboardErrors.department" class="err-msg">{{ onboardErrors.department }}</span>
          </div>
          <div v-else class="field onboarding-field">
            <label class="mono-label">Department </label>
            <input
              v-model="onboardForm.department"
              type="text"
              readonly
              :class="{ error: onboardErrors.department }"
            />
            <span v-if="onboardErrors.department" class="err-msg">{{ onboardErrors.department }}</span>
          </div>
          <div v-if="userRole === 'super admin'" ref="onboardRoleDropdownRef" class="field onboarding-field custom-select">
            <label class="mono-label">Role </label>
            <button
              type="button"
              class="custom-select__trigger"
              :class="{ error: onboardErrors.role }"
              @click.stop="onboardRoleDropdownOpen = !onboardRoleDropdownOpen"
            >
              <span class="custom-select__label">{{ onboardForm.role ? onboardForm.role.charAt(0).toUpperCase() + onboardForm.role.slice(1) : 'Select role' }}</span>
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
            <div v-if="onboardRoleDropdownOpen" class="custom-select__options">
              <button
                type="button"
                class="custom-select__option"
                :class="{ selected: !onboardForm.role }"
                @click="selectOnboardRole('')"
              >
                Select role
              </button>
              <button
                type="button"
                class="custom-select__option"
                :class="{ selected: onboardForm.role === 'user' }"
                @click="selectOnboardRole('user')"
              >
                User
              </button>
              <button
                type="button"
                class="custom-select__option"
                :class="{ selected: onboardForm.role === 'admin' }"
                @click="selectOnboardRole('admin')"
              >
                Admin
              </button>
            </div>
            <span v-if="onboardErrors.role" class="err-msg">{{ onboardErrors.role }}</span>
          </div>
          <span v-if="onboardErrors.general" class="err-msg">{{ onboardErrors.general }}</span>
          <button type="submit" class="btn btn-primary onboarding-submit" :disabled="addingUser">
            {{ addingUser ? 'Adding…' : 'Add User' }}
            <svg
              width="15"
              height="15"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
            >
              <path d="M16 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2" />
              <circle cx="8.5" cy="7" r="4" />
              <line x1="20" y1="8" x2="20" y2="14" />
              <line x1="23" y1="11" x2="17" y2="11" />
            </svg>
          </button>
        </form>
      </div>

      <nav v-if="isSuperAdmin" class="dashboard-tabs onboarding-list-tabs" role="tablist" aria-label="Onboarded users">
        <button
          role="tab"
          class="dashboard-tabs__tab"
          :class="{ active: onboardingListTab === 'users' }"
          :aria-selected="onboardingListTab === 'users'"
          @click="onboardingListTab = 'users'"
        >
          Users
        </button>
        <button
          role="tab"
          class="dashboard-tabs__tab"
          :class="{ active: onboardingListTab === 'admins' }"
          :aria-selected="onboardingListTab === 'admins'"
          @click="onboardingListTab = 'admins'"
        >
          Admins
        </button>
      </nav>

      <div v-if="!displayedOnboardingUsers.length" class="vouchers-empty card">
        <svg
          width="40"
          height="40"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="1.5"
        >
          <path d="M16 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2" />
          <circle cx="8.5" cy="7" r="4" />
          <line x1="20" y1="8" x2="20" y2="14" />
          <line x1="23" y1="11" x2="17" y2="11" />
        </svg>
        <p class="vouchers-empty__title">No {{ isSuperAdmin ? onboardingListTab : 'users' }} onboarded yet</p>
        <p class="vouchers-empty__sub">Add users above to grant them access to the system.</p>
      </div>

      <div v-else class="vouchers-table-wrap card">
        <table class="vouchers-table onboarding-table">
          <thead>
            <tr>
              <th class="text-center">S/N</th>
              <th>User</th>
              <th class="text-center">Added On</th>
              <th>Department</th>
              <th class="text-center">Status</th>
              <th class="text-center">Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(user, index) in displayedOnboardingUsers" :key="user.id" class="vouchers-table__row">
              <td class="text-center text-muted">{{ index + 1 }}</td>
              <td>
                <div class="admin-user-cell">
                  <span class="admin-user-avatar">{{ user.email.charAt(0).toUpperCase() }}</span>
                  <span class="admin-user-email">{{ user.email }}</span>
                </div>
              </td>
              <td class="text-center text-muted">{{ formatDate(user.addedAt) }}</td>
              <td class="text-muted">{{ user.department || '—' }}</td>
              <td class="text-center">
                <span
                  class="status-badge"
                  :class="
                    isUserActive(user.email) ? 'status-badge--active' : 'status-badge--pending'
                  "
                >
                  {{ isUserActive(user.email) ? 'Active' : 'Pending' }}
                </span>
              </td>
              <td class="text-center">
                <button
                  type="button"
                  class="btn btn-sm btn-destructive"
                  style="padding: 5px 10px; font-size: 12px;"
                  @click="promptDeleteUser(user)"
                  :disabled="removingUserId === user.id"
                >
                  {{ removingUserId === user.id ? 'Deleting…' : (user.role === 'admin' ? 'Delete admin' : 'Delete user') }}
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      
    </div>
  </div>
  <div v-if="showCreateDeptModal" class="modal-backdrop" @click.self="closeCreateDeptModal">
    <div
      class="modal"
      role="dialog"
      aria-modal="true"
      aria-label="Create department"
      style="max-width: 480px;"
    >
      <div class="modal-header" style="padding: 8px 24px 4px;">
        <div class="modal-header__title" style="font-size: 16px; font-weight: 700; letter-spacing: -0.04em;">
          Create department?
        </div>
        <button class="modal-close" @click="closeCreateDeptModal" aria-label="Close">✕</button>
      </div>
      <div class="modal-body">
        <div class="field">
          <label class="mono-label">Department name</label>
          <input
            v-model="newDepartment"
            type="text"
            placeholder="e.g. Marketing"
            :class="{ error: newDepartmentError }"
            @input="newDepartmentError = ''"
          />
          <span v-if="newDepartmentError" class="err-msg">{{ newDepartmentError }}</span>
        </div>
      </div>
      <div class="modal-footer" style="border-top: none;">
        <button class="btn btn-outline" @click="closeCreateDeptModal">Cancel</button>
        <button
          class="btn btn-accent"
          :disabled="!newDepartment.trim()"
          @click="handleCreateDepartment"
        >
          Create department
        </button>
      </div>
    </div>
  </div>
  <div v-if="showEditDeptModal" class="modal-backdrop" @click.self="closeEditDeptModal">
    <div
      class="modal"
      role="dialog"
      aria-modal="true"
      aria-label="Edit department"
      style="max-width: 480px;"
    >
      <div class="modal-header" style="padding: 8px 24px 4px;">
        <div class="modal-header__title" style="font-size: 16px; font-weight: 700; letter-spacing: -0.04em;">
          Edit department?
        </div>
        <button class="modal-close" @click="closeEditDeptModal" aria-label="Close">✕</button>
      </div>
      <div class="modal-body">
        <div class="field">
          <label class="mono-label">Department name</label>
          <input
            v-model="editingDeptName"
            type="text"
            placeholder="e.g. Marketing"
            :class="{ error: editingDeptError }"
            @input="editingDeptError = ''"
          />
          <span v-if="editingDeptError" class="err-msg">{{ editingDeptError }}</span>
        </div>
      </div>
      <div class="modal-footer" style="border-top: none;">
        <button class="btn btn-outline" @click="closeEditDeptModal">Cancel</button>
        <button
          class="btn btn-accent"
          :disabled="!editingDeptName.trim() || editingDeptName.trim() === editingDeptOriginal"
          @click="handleEditDepartment"
        >
          Save
        </button>
      </div>
    </div>
  </div>
  <div v-if="showDeleteDeptModal" class="modal-backdrop" @click.self="closeDeleteDeptModal">
    <div
      class="modal"
      role="dialog"
      aria-modal="true"
      aria-label="Delete department"
      style="max-width: 480px;"
    >
      <div class="modal-header" style="padding: 8px 24px 4px;">
        <div class="modal-header__title" style="font-size: 16px; font-weight: 700; letter-spacing: -0.04em;">
          Delete department?
        </div>
        <button class="modal-close" @click="closeDeleteDeptModal" aria-label="Close">✕</button>
      </div>
      <div class="modal-body">
        <p style="font-size: 18px; font-weight: 900; letter-spacing: -0.04em; margin: 0;">
          Are you sure you want to delete {{ deptToDelete }}?
          <span style="display: block; margin-top: 12px; font-size: 14px; color: var(--muted-fg); font-weight: 500;">
            This action cannot be undone.
          </span>
        </p>
      </div>
      <div class="modal-footer" style="border-top: none;">
        <button class="btn btn-outline" @click="closeDeleteDeptModal">Cancel</button>
        <button class="btn btn-destructive" @click="confirmDeleteDepartment">
          Yes, Delete
        </button>
      </div>
    </div>
  </div>
  <div v-if="showDeleteModal" class="modal-backdrop" @click.self="closeDeleteModal">
    <div
      class="modal"
      role="dialog"
      aria-modal="true"
      :aria-label="userToDelete?.role === 'admin' ? 'Delete admin confirmation' : 'Delete user confirmation'"
      style="max-width: 640px;"
    >
      <div class="modal-header" style="padding: 8px 24px 4px;">
        <div class="modal-header__title" style="font-size: 16px; font-weight: 700; letter-spacing: -0.04em;">
          Delete {{ userToDelete?.role === 'admin' ? 'admin' : 'user' }}?
        </div>
        <button class="modal-close" @click="closeDeleteModal" aria-label="Close">✕</button>
      </div>
      <div class="modal-body">
        <p style="font-size: 18px; font-weight: 900; letter-spacing: -0.04em; margin: 0;">
          Are you sure you want to delete this {{ userToDelete?.role === 'admin' ? 'admin' : 'user' }}?
          <span style="display: block; margin-top: 12px; font-size: 14px; color: var(--muted-fg); font-weight: 500;">
            {{ userToDelete?.email }} — This action cannot be undone.
          </span>
        </p>
      </div>
      <div class="modal-footer" style="border-top: none;">
        <button class="btn btn-outline" style="border-radius: 9999px;" @click="closeDeleteModal">Cancel</button>
        <button
          class="btn btn-destructive"
          :disabled="removingUserId === userToDelete?.id"
          style="border-radius: 9999px;"
          @click="confirmDeleteUser"
        >
          {{ removingUserId === userToDelete?.id ? 'Deleting…' : 'Yes, Delete' }}
        </button>
      </div>
    </div>
  </div>
  <FilePreview :show="showFilePreview" :file="previewFile" @close="showFilePreview = false" />
</template>

<script setup>
import { reactive, computed, ref, onMounted, onBeforeUnmount } from 'vue'
import FilePreview from '../components/FilePreview.vue'
import {
  allVouchers,
  loadingVouchers,
  onboardingUsers,
  addOnboardingUser,
  removeOnboardingUser,
  fetchOnboardingUsers,
  fetchCurrentUser,
  userEmail,
  userRole,
  userDepartment,
  sendInviteEmail,
} from './stores/appState'

const activeTab = ref('vouchers')
const onboardingListTab = ref('users')
const selectedVoucher = ref(null)
const addingUser = ref(false)
const removingUserId = ref('')
const showDeleteModal = ref(false)
const userToDelete = ref(null)
const loadingUsers = ref(false)
const showFilePreview = ref(false)
const previewFile = ref(null)
const deptDropdownOpen = ref(false)
const userDropdownOpen = ref(false)
const statusDropdownOpen = ref(false)
const deptDropdownRef = ref(null)
const userDropdownRef = ref(null)
const statusDropdownRef = ref(null)
const onboardDeptDropdownOpen = ref(false)
const onboardRoleDropdownOpen = ref(false)
const onboardDeptDropdownRef = ref(null)
const onboardRoleDropdownRef = ref(null)
const showCreateDeptModal = ref(false)
const newDepartment = ref('')
const newDepartmentError = ref('')
const showEditDeptModal = ref(false)
const editingDeptOriginal = ref('')
const editingDeptName = ref('')
const editingDeptError = ref('')
const showDeleteDeptModal = ref(false)
const deptToDelete = ref('')

const expandedPurposes = reactive({})

function togglePurpose(id) {
  expandedPurposes[id] = !expandedPurposes[id]
}

const onboardForm = reactive({
  email: '',
  department: userRole.value !== 'super admin' ? (userDepartment.value || '') : '',
  role: '',
})
const onboardErrors = reactive({})

onMounted(async () => {
  loadingUsers.value = true
  try {
    await fetchOnboardingUsers()
  } catch {
    onboardErrors.general = 'Could not load users. Make sure the backend is running.'
  } finally {
    loadingUsers.value = false
  }
  try {
    await fetchCurrentUser()
  } catch {
    // Department stays empty if the fetch fails
  }
  if (userRole.value !== 'super admin') {
    onboardForm.department = userDepartment.value || ''
  }
  window.addEventListener('click', closeDropdowns)
})

const adminFilter = reactive({ dept: '', user: '', status: '' })

const isSuperAdmin = computed(() => userRole.value === 'super admin')

const displayedOnboardingUsers = computed(() => {
  if (!isSuperAdmin.value) return onboardingUsers.value
  if (onboardingListTab.value === 'admins') {
    return onboardingUsers.value.filter((user) => user.role === 'admin')
  }
  return onboardingUsers.value.filter((user) => user.role === 'user')
})

const adminBaseVouchers = computed(() => {
  const onboardedEmails = new Set(onboardingUsers.value.map((user) => user.email.toLowerCase()))
  return [...allVouchers.value].reverse().filter((voucher) => {
    const submitted = String(voucher.submittedBy || '').toLowerCase()
    const current = userEmail.value.toLowerCase()
    if (submitted === current) return false
    if (!isSuperAdmin.value) {
      return onboardedEmails.has(submitted)
    }
    return true
  })
})
const adminDepts = computed(() =>
  [...new Set(adminBaseVouchers.value.map((voucher) => voucher.department).filter(Boolean))].sort(),
)
const onboardDepts = ref(['firstDep', 'secondDep', 'thirdDep', 'fourthDep', 'fifthDep'])
const adminUsers = computed(() =>
  [...new Set(adminBaseVouchers.value.map((voucher) => voucher.submittedBy).filter(Boolean))].sort(),
)
const adminStatuses = computed(() =>
  [...new Set(adminBaseVouchers.value.map((voucher) => (voucher.status || 'Pending').toLowerCase()).filter(Boolean))].sort(),
)

function openFilePreview(doc) {
  previewFile.value = doc
  showFilePreview.value = true
}

function formatStatus(status) {
  return status ? status.charAt(0).toUpperCase() + status.slice(1) : 'All Statuses'
}

function formatDate(date) {
  return date ? new Date(date).toLocaleDateString('en-CA') : '—'
}

function selectDept(dept) {
  adminFilter.dept = dept
  deptDropdownOpen.value = false
}

function selectUser(user) {
  adminFilter.user = user
  userDropdownOpen.value = false
}

function selectStatus(status) {
  adminFilter.status = status
  statusDropdownOpen.value = false
}

function selectOnboardDept(dept) {
  onboardForm.department = dept
  delete onboardErrors.department
  onboardDeptDropdownOpen.value = false
}

function selectOnboardRole(role) {
  onboardForm.role = role
  delete onboardErrors.role
  onboardRoleDropdownOpen.value = false
}

function openCreateDeptModal() {
  newDepartment.value = ''
  newDepartmentError.value = ''
  showCreateDeptModal.value = true
}

function closeCreateDeptModal() {
  showCreateDeptModal.value = false
  newDepartment.value = ''
  newDepartmentError.value = ''
}

function handleCreateDepartment() {
  const dept = newDepartment.value.trim()
  if (!dept) return
  if (onboardDepts.value.includes(dept)) {
    newDepartmentError.value = 'Department already exists'
    return
  }
  onboardDepts.value.push(dept)
  onboardDepts.value.sort()
  closeCreateDeptModal()
}

function openEditDeptModal(dept) {
  editingDeptOriginal.value = dept
  editingDeptName.value = dept
  editingDeptError.value = ''
  showEditDeptModal.value = true
}

function closeEditDeptModal() {
  showEditDeptModal.value = false
  editingDeptOriginal.value = ''
  editingDeptName.value = ''
  editingDeptError.value = ''
}

function handleEditDepartment() {
  const name = editingDeptName.value.trim()
  if (!name) {
    editingDeptError.value = 'Department name is required'
    return
  }
  if (name !== editingDeptOriginal.value && onboardDepts.value.includes(name)) {
    editingDeptError.value = 'Department already exists'
    return
  }
  const index = onboardDepts.value.indexOf(editingDeptOriginal.value)
  if (index !== -1) {
    onboardDepts.value[index] = name
    onboardDepts.value.sort()
  }
  if (onboardForm.department === editingDeptOriginal.value) {
    onboardForm.department = name
  }
  closeEditDeptModal()
}

function openDeleteDeptModal(dept) {
  deptToDelete.value = dept
  showDeleteDeptModal.value = true
}

function closeDeleteDeptModal() {
  showDeleteDeptModal.value = false
  deptToDelete.value = ''
}

function confirmDeleteDepartment() {
  if (!deptToDelete.value) return
  const index = onboardDepts.value.indexOf(deptToDelete.value)
  if (index !== -1) {
    onboardDepts.value.splice(index, 1)
  }
  if (onboardForm.department === deptToDelete.value) {
    onboardForm.department = ''
  }
  closeDeleteDeptModal()
}

function closeDropdowns(event) {
  if (deptDropdownRef.value && !deptDropdownRef.value.contains(event.target)) {
    deptDropdownOpen.value = false
  }
  if (userDropdownRef.value && !userDropdownRef.value.contains(event.target)) {
    userDropdownOpen.value = false
  }
  if (statusDropdownRef.value && !statusDropdownRef.value.contains(event.target)) {
    statusDropdownOpen.value = false
  }
  if (onboardDeptDropdownRef.value && !onboardDeptDropdownRef.value.contains(event.target)) {
    onboardDeptDropdownOpen.value = false
  }
  if (onboardRoleDropdownRef.value && !onboardRoleDropdownRef.value.contains(event.target)) {
    onboardRoleDropdownOpen.value = false
  }
}

onBeforeUnmount(() => {
  window.removeEventListener('click', closeDropdowns)
})

const filteredAdminVouchers = computed(() => {
  return adminBaseVouchers.value.filter(
    (voucher) =>
      (!adminFilter.dept || voucher.department === adminFilter.dept) &&
      (!adminFilter.user || voucher.submittedBy === adminFilter.user) &&
      (!adminFilter.status ||
        (voucher.status || 'Pending').toLowerCase() === adminFilter.status),
  )
})


const activeVoucherEmails = computed(
  () => new Set(allVouchers.value.map((voucher) => voucher.submittedBy)),
)

function isUserActive(email) {
  return activeVoucherEmails.value.has(email)
}

const pendingOnboardingCount = computed(
  () => onboardingUsers.value.filter((user) => !isUserActive(user.email)).length,
)

function isOnboardEmail(v) {
  return /^[^\s@]+@(getpayedmail\.com|gmail\.com)$/.test(v)
}

function generateStrongPassword() {
  const length = 8
  const charset = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!@#$%^&*'
  let password = ''
  const array = new Uint32Array(length)
  crypto.getRandomValues(array)
  for (let i = 0; i < length; i++) {
    password += charset[array[i] % charset.length]
  }
  return password
}


async function handleAddUser() {
  delete onboardErrors.email
  delete onboardErrors.department
  delete onboardErrors.role
  delete onboardErrors.general

  if (!isOnboardEmail(onboardForm.email)) {
    onboardErrors.email = 'Email must be a @getpayedmail.com or @gmail.com address'
  }
  if (!onboardForm.department) {
    onboardErrors.department = 'Please select a department'
  }
  if (userRole.value === 'super admin' && !onboardForm.role) {
    onboardErrors.role = 'Please select a role'
  }
  if (Object.keys(onboardErrors).length) return

  const password = generateStrongPassword()
  addingUser.value = true
  try {
    await addOnboardingUser(onboardForm.email, password, userEmail.value, onboardForm.department, onboardForm.role)
    await sendInviteEmail(onboardForm.email, password, userEmail.value)
    onboardForm.email = ''
    onboardForm.department = ''
    onboardForm.role = ''
  } catch (error) {
    const message = error instanceof Error ? error.message : 'Failed to add user'
    if (message.includes('already been added')) {
      onboardErrors.email = message
    } else {
      onboardErrors.general = message
    }
  } finally {
    addingUser.value = false
  }
}

function promptDeleteUser(user) {
  userToDelete.value = user
  showDeleteModal.value = true
}

function closeDeleteModal() {
  showDeleteModal.value = false
  userToDelete.value = null
}

async function confirmDeleteUser() {
  if (!userToDelete.value) return
  try {
    await handleRemoveUser(userToDelete.value.id)
  } finally {
    closeDeleteModal()
  }
}

async function handleRemoveUser(id) {
  removingUserId.value = id
  try {
    await removeOnboardingUser(id)
  } catch (error) {
    onboardErrors.general = error instanceof Error ? error.message : 'Failed to remove user'
  } finally {
    removingUserId.value = ''
  }
}
</script>

<style scoped>
.admin-all-vouchers .vouchers-table th,
.admin-all-vouchers .vouchers-table td {
  white-space: normal;
  overflow-wrap: break-word;
  padding: 8px 10px;
}

.admin-all-vouchers .admin-user-email,
.admin-all-vouchers .voucher-purpose {
  white-space: normal;
  overflow-wrap: break-word;
}

.admin-all-vouchers .admin-user-email {
  max-width: 120px;
}

.admin-all-vouchers .voucher-purpose {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  line-clamp: 2;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 140px;
  white-space: normal;
  cursor: pointer;
  color: #000;
}

.admin-all-vouchers .voucher-purpose.expanded {
  -webkit-line-clamp: unset;
  line-clamp: unset;
  display: block;
}

.admin-all-vouchers .vouchers-table th:nth-child(3),
.admin-all-vouchers .vouchers-table td:nth-child(3) {
  text-align: center;
}

.onboarding-field {
  min-width: 160px;
}

.onboarding-field select {
  appearance: none;
  -webkit-appearance: none;
  width: 100%;
  padding: 10px 38px 10px 14px;
  border-radius: var(--radius);
  border: 1px solid var(--border);
  background-color: var(--input-bg);
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='14' height='14' viewBox='0 0 24 24' fill='none' stroke='%237a7265' stroke-width='3' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpolyline points='6 9 12 15 18 9'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 14px center;
  background-size: 14px;
  font-family: var(--font-body);
  font-size: 14px;
  color: var(--fg);
  cursor: pointer;
  outline: none;
}

.onboarding-field select:focus {
  border-color: color-mix(in srgb, var(--primary) 50%, transparent);
  box-shadow: 0 0 0 3px color-mix(in srgb, var(--primary) 12%, transparent);
}

.onboarding-field select.error {
  border-color: color-mix(in srgb, var(--destructive) 60%, transparent);
}

.custom-select__trigger.error {
  border-color: color-mix(in srgb, var(--destructive) 60%, transparent);
}

.vouchers-loading {
  display: grid;
  place-items: center;
  min-height: 60vh;
  color: #000;
  font-size: 18px;
  font-weight: 700;
  letter-spacing: -0.04em;
}

.onboarding-list-tabs {
  width: 100%;
}

.onboarding-list-tabs .dashboard-tabs__tab {
  flex: 1;
  justify-content: center;
}

.btn-destructive {
  background: #e60000;
  color: #fff;
}

.btn-destructive:hover:not(:disabled) {
  opacity: 0.88;
}

.btn-sm {
  padding: 6px 12px;
  font-size: 13px;
}

.custom-select__option--row {
  display: flex;
  align-items: center;
  padding: 0;
  overflow: hidden;
}

.custom-select__option-label {
  flex: 1;
  padding: 10px 14px;
  border: none;
  background: transparent;
  color: inherit;
  font: inherit;
  text-align: left;
  cursor: pointer;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  min-width: 0;
}

.custom-select__option-actions {
  display: flex;
  align-items: center;
  gap: 4px;
  padding-right: 8px;
  flex-shrink: 0;
}

.icon-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 26px;
  height: 26px;
  border: none;
  border-radius: var(--radius);
  background: transparent;
  color: var(--muted-fg);
  cursor: pointer;
  padding: 0;
}

.icon-btn:hover:not(:disabled) {
  background: var(--muted);
  color: var(--fg);
}

.icon-btn--destructive:hover:not(:disabled) {
  background: color-mix(in srgb, var(--destructive) 10%, transparent);
  color: var(--destructive);
}
</style>
