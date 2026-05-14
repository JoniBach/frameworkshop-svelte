<script lang="ts">
  type RegisteredUser = {
    id: string
    email: string
    createdAt: string
  }

  let email = ''
  let password = ''
  let confirmPassword = ''
  let touched = { email: false, password: false, confirmPassword: false }
  let submitted = false
  let isSubmitting = false
  let registeredUser: RegisteredUser | null = null
  let error: string | null = null

  const emailError = () => {
    if (!touched.email && !submitted) return ''
    if (!email) return 'Email is required.'
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) return 'Please enter a valid email address.'
    return ''
  }

  const passwordError = () => {
    if (!touched.password && !submitted) return ''
    if (!password) return 'Password is required.'
    if (password.length < 9) return 'Password must be at least 9 characters.'
    return ''
  }

  const confirmPasswordError = () => {
    if (!touched.confirmPassword && !submitted) return ''
    if (!confirmPassword) return 'Please confirm your password.'
    if (confirmPassword !== password) return 'Passwords must match.'
    return ''
  }

  function onBlur(field: 'email' | 'password' | 'confirmPassword') {
    touched = { ...touched, [field]: true }
  }

  async function onSubmit() {
    submitted = true
    touched = { email: true, password: true, confirmPassword: true }

    if (emailError() || passwordError() || confirmPasswordError()) return

    try {
      isSubmitting = true
      const response = await fetch('http://localhost:3000/auth/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password }),
      })

      const result = await response.json().catch(() => ({ error: 'unknown' }))

      if (!response.ok) {
        throw new Error(result.error || `Server responded ${response.status}`)
      }

      registeredUser = result.user
      error = null
      password = ''
      confirmPassword = ''
    } catch (err) {
      console.error('Error registering user', err)
      registeredUser = null
      error = (err as Error).message
      password = ''
      confirmPassword = ''
    } finally {
      isSubmitting = false
    }
  }
</script>

<svelte:head>
  <title>Register</title>
</svelte:head>

<section class="input-example">
  <h1>Register</h1>

  <p>Create a user by sending an email and password to the Deno backend. The backend hashes the password before storing it in Deno KV.</p>

  {#if error}
    <p class="error">Registration failed: {error}</p>
  {/if}

  {#if registeredUser}
    <div>
      <h2>Registered user</h2>
      <p><strong>Email:</strong> {registeredUser.email}</p>
      <p><small>Created at {new Date(registeredUser.createdAt).toLocaleTimeString()}</small></p>
    </div>
  {/if}

  <form on:submit|preventDefault={onSubmit} novalidate>
    <label>
      Email
      <input type="email" bind:value={email} on:blur={() => onBlur('email')} placeholder="you@example.com" />
    </label>
    <div class="validation">
      {#if emailError()}
        <small class="error">{emailError()}</small>
      {/if}
    </div>

    <label>
      Password
      <input type="password" bind:value={password} on:blur={() => onBlur('password')} placeholder="At least 9 characters" />
    </label>
    <div class="validation">
      {#if passwordError()}
        <small class="error">{passwordError()}</small>
      {/if}
    </div>

    <label>
      Confirm password
      <input type="password" bind:value={confirmPassword} on:blur={() => onBlur('confirmPassword')} placeholder="Repeat password" />
    </label>
    <div class="validation">
      {#if confirmPasswordError()}
        <small class="error">{confirmPasswordError()}</small>
      {/if}
    </div>

    <button type="submit" disabled={isSubmitting}>Register</button>
  </form>
</section>
