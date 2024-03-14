<script>
  import Nav from "$lib/Nav.svelte";
  import "../app.pcss";
</script>

<div class="flex flex-col h-screen bg-background">
  <Nav />
  <slot />
</div>
