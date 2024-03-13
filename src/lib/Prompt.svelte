<script lang="ts">
  import { onMount } from "svelte";
  import { Button } from "$lib/components/ui/button/index.js";
  import { prompts } from "./prompts";

  let randomPrompt: string = " ";

  function generateUniquePrompt() {
    let newPrompt: string;
    do {
      newPrompt = prompts[Math.floor(Math.random() * prompts.length)];
    } while (newPrompt === randomPrompt);
    randomPrompt = newPrompt;
  }

  onMount(() => {
    generateUniquePrompt();
  });
</script>

<Button
  variant="outline"
  class="w-full rounded-none"
  on:click={generateUniquePrompt}
>
  {randomPrompt}
</Button>
