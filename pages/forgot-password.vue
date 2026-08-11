<template>
  <div class="login-wrap">
    <div class="login-card">
      <h1 class="forgot-title">Forgot Password?</h1>
      <p class="forgot-sub">Enter your email and we'll send you a reset link.</p>

      <form class="login-form" @submit.prevent="handleSubmit" novalidate>
        <div class="email-field">
          <span class="email-icon">
            <svg
              width="18"
              height="18"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
            >
              <rect x="2" y="4" width="20" height="16" rx="2" />
              <path d="m22 7-8.97 5.7a1.94 1.94 0 0 1-2.06 0L2 7" />
            </svg>
          </span>
          <input
            v-model="email"
            type="email"
            placeholder="Enter your email"
            :disabled="loading"
            :class="{ error: error }"
          />
        </div>

        <span v-if="error" class="err-msg">{{ error }}</span>
        <span v-if="message" class="success-msg">{{ message }}</span>

        <button
          type="submit"
          class="btn btn-primary login-submit"
          :disabled="loading"
        >
          {{ loading ? 'Sending...' : 'Send reset password link' }}
        </button>
      </form>

      <NuxtLink to="/login" class="forgot-link">Back to Login</NuxtLink>
    </div>
  </div>
</template>

<script setup>
definePageMeta({ layout: false })
import { ref } from 'vue'
import { API_BASE } from '~/composables/appState'

const email = ref('')
const loading = ref(false)
const error = ref('')
const message = ref('')

async function handleSubmit() {
  error.value = ''
  message.value = ''

  if (!email.value.trim()) {
    error.value = 'Email is required'
    return
  }

  loading.value = true
  try {
    const res = await fetch(`${API_BASE}/api/auth/forgot-password`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email: email.value.trim().toLowerCase() }),
    })
    const data = await res.json()

    if (!res.ok) {
      error.value = data.error || 'Failed to send reset link'
      return
    }

    message.value = 'If an account with that email exists, a reset link has been sent.'
    email.value = ''
  } catch {
    error.value = 'Could not reach the server. Make sure the backend is running.'
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.forgot-title {
  font-size: 28px;
  font-weight: 700;
  color: var(--fg);
  margin-bottom: 8px;
  text-align: center;
}

.forgot-sub {
  font-size: 14px;
  color: var(--muted-fg);
  margin-bottom: 28px;
  text-align: center;
}

.email-field {
  position: relative;
}

.email-icon {
  position: absolute;
  left: 14px;
  top: 50%;
  transform: translateY(-50%);
  color: var(--muted-fg);
  display: flex;
  align-items: center;
  pointer-events: none;
}

.email-field input {
  width: 100%;
  padding: 10px 14px 10px 42px;
  border-radius: var(--radius);
  border: 1px solid var(--border);
  background: var(--input-bg);
  font-size: 14px;
  color: var(--fg);
  outline: none;
  transition:
    border-color 0.15s,
    box-shadow 0.15s;
}

.email-field input:focus {
  border-color: color-mix(in srgb, var(--primary) 50%, transparent);
  box-shadow: 0 0 0 3px color-mix(in srgb, var(--primary) 12%, transparent);
}

.email-field input.error {
  border-color: color-mix(in srgb, var(--destructive) 60%, transparent);
}

.forgot-link {
  display: block;
  text-align: center;
  margin-top: 20px;
  font-size: 13px;
  color: var(--primary);
  font-weight: 500;
}

.forgot-link:hover {
  color: var(--accent);
}

.success-msg {
  font-size: 13px;
  color: #059669;
  text-align: center;
}
</style>
