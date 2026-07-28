<template>
  <div v-if="!isLoggedIn" class="login-wrap">
    <div class="login-card">
      <div class="login-card__logo">
        <svg
          width="22"
          height="22"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
        >
          <rect x="2" y="7" width="20" height="14" rx="2" />
          <path d="M16 7V5a2 2 0 0 0-2-2h-4a2 2 0 0 0-2 2v2" />
        </svg>
      </div>
      <p class="login-card__company">Getpayed Technology Solutions Ltd.</p>
      <h1 class="login-card__title serif">Sign in to your account</h1>
      <p class="login-card__sub">Petty Cash Voucher System</p>

      <form class="login-form" @submit.prevent="handleLogin" novalidate>
        <FormField
          v-model="loginForm.email"
          label="Email Address"
          type="email"
          placeholder="user@getpayedmail.com or @gmail.com"
          :error="loginErrors.email"
          :disabled="loggingIn"
          @input="delete loginErrors.email"
        />

        <FormField
          v-model="loginForm.password"
          label="Password"
          :type="showPassword ? 'text' : 'password'"
          placeholder="Enter your password"
          :error="loginErrors.password"
          :disabled="loggingIn"
          @input="delete loginErrors.password"
        >
          <template #suffix>
            <button
              type="button"
              class="pwd-toggle"
              @click="showPassword = !showPassword"
              :aria-label="showPassword ? 'Hide password' : 'Show password'"
            >
              <svg
                v-if="!showPassword"
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
              <svg
                v-else
                width="15"
                height="15"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
              >
                <path
                  d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94"
                />
                <path d="M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19" />
                <line x1="1" y1="1" x2="23" y2="23" />
              </svg>
            </button>
          </template>
        </FormField>

        <span v-if="loginErrors.general" class="err-msg login-general-error">{{
          loginErrors.general
        }}</span>

        <button
          type="submit"
          class="btn btn-primary login-submit"
          :disabled="loggingIn"
        >
          {{ loggingIn ? 'Signing in...' : 'Sign in' }}
        </button>
      </form>
    </div>
  </div>
</template>

<script setup>
import { onMounted, reactive, ref } from 'vue'
import { useRouter } from 'vue-router'
import FormField from '../components/FormField.vue'
import { API_BASE, fetchCurrentUser, isLoggedIn, loginUser } from './stores/appState'

const router = useRouter()

function isLoginEmail(v) {
  return /^[^\s@]+@(getpayedmail\.com|gmail\.com)$/.test(v)
}

const showPassword = ref(false)
const loggingIn = ref(false)
const loginForm = reactive({ email: '', password: '' })
const loginErrors = reactive({})

onMounted(() => {
  if (isLoggedIn.value) router.replace({ name: 'form' })
})

async function handleLogin() {
  delete loginErrors.email
  delete loginErrors.password
  delete loginErrors.general

  if (!isLoginEmail(loginForm.email)) {
    loginErrors.email = 'Email must be a @getpayedmail.com or @gmail.com address'
  }
  if (!loginForm.password) {
    loginErrors.password = 'Password is required'
  }

  if (Object.keys(loginErrors).length) return

  loggingIn.value = true
  try {
    const res = await fetch(`${API_BASE}/api/auth/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        email: loginForm.email,
        password: loginForm.password,
      }),
    })

    const data = await res.json()
    if (!res.ok) {
      loginErrors.general = data.error || 'Login failed'
      return
    }

    loginUser(data.email, data.role)
    await fetchCurrentUser()
    router.replace(data.role === 'admin' || data.role === 'super admin' ? { name: 'admin' } : { name: 'form' })
  } catch {
    loginErrors.general = 'Could not reach the server. Make sure the backend is running.'
  } finally {
    loggingIn.value = false
  }
}
</script>
