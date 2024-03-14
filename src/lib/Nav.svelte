<script>
  import { Heart, ListMusic, Waves } from "lucide-svelte";
  import TransFlag from "./TransFlag.svelte";
  import * as Tooltip from "$lib/components/ui/tooltip";
  import Button from "./components/ui/button/button.svelte";

  import { onMount } from "svelte";
  import { cn } from "./utils";

  let currentPath = "";

  onMount(() => {
    currentPath = window.location.pathname;
  });
</script>

<header
  class="flex items-center border-b border-border/40 h-14 px-8 justify-between"
>
  <div class="flex gap-4 items-center">
    <TransFlag class="w-6 h-6" />
    <h1 class="font-bold text-lg">Trans Voice</h1>
    <a
      class={cn(
        "flex transition-colors hover:text-foreground/80 text-foreground/60 flex-row gap-2",
        currentPath === "/" && "text-foreground/80"
      )}
      href="/"
    >
      <Waves /> Pitch
    </a>
    <a
      class={cn(
        "flex transition-colors hover:text-foreground/80 text-foreground/60 flex-row gap-2",
        currentPath === "/tone" && "text-foreground/80"
      )}
      href="/tone"
    >
      <ListMusic /> Tone
    </a>
  </div>
  <Tooltip.Root>
    <Tooltip.Trigger>
      <Button variant="ghost" class="flex flex-row gap-2"
        >Support us!<Heart class="size-4 stroke-[#F7A8B8]" /></Button
      >
    </Tooltip.Trigger>
    <Tooltip.Content>
      <p>Support us!</p>
    </Tooltip.Content>
  </Tooltip.Root>
</header>
