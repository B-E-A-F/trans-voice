<script lang="ts">
  // @ts-nocheck
  import { PitchDetector } from "pitchy";
  import { onMount } from "svelte";

  let pitch = 0;
  export let audioContext;

  function updatePitch(
    analyserNode: AnalyserNode,
    detector,
    input,
    sampleRate
  ) {
    analyserNode.getFloatTimeDomainData(input);
    const pitchDiscovery = detector.findPitch(input, sampleRate);

    pitch = Math.round(pitchDiscovery[0] * 10) / 10;

    window.setTimeout(
      () => updatePitch(analyserNode, detector, input, sampleRate),
      100
    );
  }

  onMount(() => {
    const analyserNode = audioContext.createAnalyser();

    navigator.mediaDevices.getUserMedia({ audio: true }).then((stream) => {
      audioContext.createMediaStreamSource(stream).connect(analyserNode);
      const detector = PitchDetector.forFloat32Array(analyserNode.fftSize);
      detector.minVolumeDecibels = -10;
      const input = new Float32Array(detector.inputLength);
      updatePitch(analyserNode, detector, input, audioContext.sampleRate);
    });
  });
</script>

<div class="text-center text-8xl">
  <p>{pitch} Hz</p>
</div>
