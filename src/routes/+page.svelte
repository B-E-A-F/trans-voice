<script lang="ts">
  import Nav from "$lib/Nav.svelte";
  import Prompt from "$lib/Prompt.svelte";
</script>

<div class="flex flex-col h-full">
  <!-- Nav Bar -->
  <Nav />

  <!-- Graph Container -->
  <div class="flex flex-grow bg-neutral-800" />

  <!-- Prompt -->
  <Prompt />
</div>
