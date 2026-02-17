<script lang="ts">
  import TextInput from '$lib/TextInput.svelte'
  import './input-example.css'
  let name: string = ''
</script>

<svelte:head>
  <title>Input Example</title>
</svelte:head>

<section class="input-example">
  <h1>Input Example</h1>
  <TextInput bind:value={name} label="Your name" placeholder="Type a name" />
  <p>Current value: {#if name}{name}{:else}<em>(empty)</em>{/if}</p>
</section>
