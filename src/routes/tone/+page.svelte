<script lang="ts">
  import { Slider } from "$lib/components/ui/slider";
  import Button from "$lib/components/ui/button/button.svelte";
  import { Pause, Play } from "lucide-svelte";
  import { onMount } from "svelte";

  let audioContext: AudioContext;
  let oscillator: OscillatorNode;
  let frequency: number[] = [440];
  let isPlaying = false;

  onMount(() => {
    // Create audio context
    audioContext = new window.AudioContext();

    // Create an oscillator node
    oscillator = audioContext.createOscillator();

    // Set oscillator type (sine, square, sawtooth, triangle)
    oscillator.type = "sine";

    // Set frequency (Hz)
    oscillator.frequency.value = frequency[0]; // A440

    // Start the oscillator
    oscillator.start();
  });

  function toggleOscillator() {
    if (isPlaying) {
      oscillator.disconnect();
    } else {
      oscillator.connect(audioContext.destination);
    }
    isPlaying = !isPlaying;
  }

  $: if (oscillator) oscillator.frequency.value = frequency[0];
</script>

<div class="flex">
  <p class="text-lg">{frequency[0]} Hz</p>
  <div class="flex flex-1 justify-start">
    <Slider bind:value={frequency} max={440} step={1} />
  </div>
  <Button
    variant="ghost"
    class="flex flex-row gap-2"
    on:click={toggleOscillator}
  >
    {#if isPlaying}
      <Pause />
    {:else}
      <Play />
    {/if}
  </Button>
</div>
