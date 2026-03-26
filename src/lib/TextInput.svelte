<script lang="ts">
  import { createEventDispatcher } from 'svelte'
  export let label: string | undefined
  export let value: string = ''
  export let placeholder: string | undefined
  export let showClear: boolean = true

  const dispatch = createEventDispatcher()

  function onInput(e: Event) {
    const v = (e.target as HTMLInputElement).value
    // dispatch event matching the prop name to support bind:value
    dispatch('value', v)
  }

  function onBlur() {
    dispatch('blur')
  }

  function clear() {
    dispatch('value', '')
  }
</script>

<div class="text-input">
  {#if label}
    <label class="text-input__label">{label}</label>
  {/if}
  <div class="text-input__row">
    <input
      class="text-input__field"
      value={value}
      on:input={onInput}
      on:blur={onBlur}
      placeholder={placeholder}
    />
    {#if showClear && value}
      <button type="button" class="text-input__clear" on:click={clear} aria-label="Clear input">×</button>
    {/if}
  </div>
</div>
