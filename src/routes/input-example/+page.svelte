<script lang="ts">
  import { onMount } from 'svelte'
  import './input-example.css'

  type SavedEmail = {
    id: string
    timestamp: string
    input: {
      email: string
    }
  }

  let email: string = ''
  let touched = false
  let submitted = false
  let savedEmail: SavedEmail | null = null
  let error: string | null = null

  const emailError = () => {
    if (!touched && !submitted) return ''
    if (!email) return 'Email is required.'
    const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    if (!re.test(email)) return 'Please enter a valid email address.'
    return ''
  }

  const fetchEmail = async () => {
    try {
      const res = await fetch('http://localhost:3000/input-example')
      if (!res.ok) throw new Error(`GET failed ${res.status}`)
      const json = await res.json() as { status: string; data: SavedEmail | null }
      if (json.status === 'ok' && json.data) {
        savedEmail = json.data
        email = json.data.input.email
      } else {
        savedEmail = null
      }
      error = null
    } catch (err) {
      console.error('Unable to fetch saved email', err)
      error = (err as Error).message
    }
  }

  onMount(() => {
    fetchEmail()
  })

  function onBlur() {
    touched = true
  }

  async function onSubmit(e: Event) {
    e.preventDefault()
    submitted = true
    touched = true
    const err = emailError()
    console.log('onSubmit called', { email, err })

    if (err) {
      console.warn('Validation stopped submit', err)
      return
    }

    try {
      const response = await fetch('http://localhost:3000/input-example', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email }),
      })

      if (!response.ok) {
        const payload = await response.json().catch(() => ({ error: 'unknown' }))
        throw new Error(`Server responded ${response.status}: ${JSON.stringify(payload)}`)
      }

      const result = await response.json()
      savedEmail = result.saved
      alert(`Saved to Deno KV: ${JSON.stringify(result.saved)}`)
      console.log('Deno API result:', result)
      await fetchEmail()
    } catch (err) {
      console.error('Error saving email to Deno KV', err)
      alert(`Failed to save email to backend: ${(err as Error).message}`)
    }
  }
</script>

<svelte:head>
  <title>Input Example</title>
</svelte:head>

<section class="input-example">
  <h1>Email Input Example (Svelte)</h1>

  <div style="margin-bottom: 16px;">
    <h2>Saved email from Deno KV</h2>

    {#if error}
      <p class="error">Failed to load: {error}</p>
    {:else if !savedEmail}
      <p>No saved email yet.</p>
    {:else}
      <p><strong>Saved email:</strong> {savedEmail.input.email}</p>
      <p><small>Saved at {new Date(savedEmail.timestamp).toLocaleTimeString()}</small></p>
    {/if}
  </div>

  <form on:submit|preventDefault={onSubmit} novalidate>
    <label>
      Email
      <input
        type="email"
        bind:value={email}
        on:blur={onBlur}
        placeholder="you@example.com"
      />
    </label>

    <div class="validation">
      {#if emailError()}
        <small class="error">{emailError()}</small>
      {/if}
    </div>

    <button type="submit">Submit</button>
  </form>
</section>
