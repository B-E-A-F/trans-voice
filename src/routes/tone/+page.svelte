<script lang="ts">
  import { Slider } from "$lib/components/ui/slider";
  import Button from "$lib/components/ui/button/button.svelte";
  import { Pause, Play, Volume2, VolumeX } from "lucide-svelte";
  import { onMount } from "svelte";
  import MicFrequency from "./MicFrequency.svelte";

  let audioContext: AudioContext;
  let oscillator: OscillatorNode;
  let frequency: number[] = [440];
  let isPlaying = false;
  let gain: GainNode;
  let gainValue = [0.5];

  onMount(() => {
    // Create audio context
    audioContext = new window.AudioContext();

    // Create an oscillator node
    oscillator = audioContext.createOscillator();

    // Create a gain node
    gain = audioContext.createGain();

    // Set oscillator type (sine, square, sawtooth, triangle)
    oscillator.type = "sine";

    // Set frequency (Hz)
    oscillator.frequency.value = frequency[0]; // A440

    // Connect gain node to audio context destination
    gain.connect(audioContext.destination);

    // Start the oscillator
    oscillator.start();
  });

  function toggleOscillator() {
    if (isPlaying) {
      oscillator.disconnect();
    } else {
      oscillator.connect(gain);
    }
    isPlaying = !isPlaying;
  }

  // Dynamic frequency change
  $: if (oscillator) oscillator.frequency.value = frequency[0];
  // Dynamic gain change
  $: if (gain) gain.gain.value = gainValue[0];
</script>

<div class="flex flex-col gap-4 flex-grow justify-center">
  <p class="text-8xl text-center">{frequency[0]} Hz</p>
  <div class="flex justify-center">
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
    <div class="flex flex-1 max-w-80">
      <Slider bind:value={frequency} max={440} step={1} />
    </div>
  </div>
  <div class="flex flex-col justify-center">
    {#if audioContext}
      <MicFrequency {audioContext} />
    {/if}
    <div class="flex justify-center">
      <Button
        variant="ghost"
        class="flex flex-row gap-2"
        on:click={() => (gainValue[0] = 0)}
      >
        {#if gainValue[0] > 0}
          <Volume2 />
        {:else}
          <VolumeX />
        {/if}
      </Button>
      <div class="flex flex-1 max-w-80">
        <Slider bind:value={gainValue} min={0} max={1} step={0.01} />
      </div>
    </div>
  </div>
</div>
