<script lang="ts">
  import TextInput from '$lib/TextInput.svelte'
  import './input-example.css'

  let email: string = ''
  let touched = false
  let submitted = false

  const emailError = () => {
    if (!touched && !submitted) return ''
    if (!email) return 'Email is required.'
    const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    if (!re.test(email)) return 'Please enter a valid email address.'
    return ''
  }

  function onBlur() {
    touched = true
  }

  function onSubmit(e: Event) {
    e.preventDefault()
    submitted = true
    touched = true
    const err = emailError()
    if (!err) {
      alert(`Submitted: ${email}`)
      console.log('Submitted:', email)
    }
  }
</script>

<svelte:head>
  <title>Input Example</title>
</svelte:head>

<section class="input-example">
  <h1>Email Input Example (Svelte)</h1>

  <form on:submit|preventDefault={onSubmit} novalidate>
    <TextInput bind:value={email} label="Email" placeholder="you@example.com" on:blur={onBlur} />

    <div class="validation">
      {#if emailError()}
        <small class="error">{emailError()}</small>
      {/if}
    </div>

    <button type="submit">Submit</button>
  </form>
</section>
