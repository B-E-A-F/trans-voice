<script lang="ts">
  import type { HTMLAttributes } from "svelte/elements";

  type $$Props = HTMLAttributes<SVGElement>;
</script>

<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 256 256" {...$$restProps}>
  <rect width="256" height="256" fill="none" />
  <!-- Light Blue Stripe (Top) -->
  <rect x="0" y="0" width="256" height="51.2" fill="#55CDFC" />
  <!-- Pink Stripe -->
  <rect x="0" y="51.2" width="256" height="51.2" fill="#F7A8B8" />
  <!-- White Stripe -->
  <rect x="0" y="102.4" width="256" height="51.2" fill="#FFFFFF" />
  <!-- Pink Stripe -->
  <rect x="0" y="153.6" width="256" height="51.2" fill="#F7A8B8" />
  <!-- Light Blue Stripe (Bottom) -->
  <rect x="0" y="204.8" width="256" height="51.2" fill="#55CDFC" />
</svg>
