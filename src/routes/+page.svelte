<script lang="ts">
  import Nav from "$lib/Nav.svelte"
  import Prompt from "$lib/Prompt.svelte"
  import VoiceGraph from "$lib/components/voice-graph.svelte"
</script>

<div class="flex flex-col h-full">
  <!-- Nav Bar -->
  <Nav />

  <!-- Graph Container -->
  <div class="flex flex-grow bg-background">
    <div class="flex flex-col items-center justify-center w-full h-full">
      <VoiceGraph />
    </div>
  </div>

  <!-- Prompt -->
  <Prompt />
</div>
