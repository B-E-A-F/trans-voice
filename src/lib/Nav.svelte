<script>
  import { Heart } from "lucide-svelte";
  import TransFlag from "./TransFlag.svelte";
  import * as Tooltip from "$lib/components/ui/tooltip";
  import Button from "./components/ui/button/button.svelte";
</script>

<header
  class="flex items-center border-b border-border/40 h-14 px-8 justify-between"
>
  <div class="flex gap-4 items-center">
    <TransFlag class="w-6 h-6" />
    <h1 class="font-bold text-lg">Trans Voice</h1>
  </div>
  <Tooltip.Root>
    <Tooltip.Trigger>
      <Button variant="ghost"><Heart class="size-4 stroke-[#F7A8B8]" /></Button>
    </Tooltip.Trigger>
    <Tooltip.Content>
      <p>Sponsor us!</p>
    </Tooltip.Content>
  </Tooltip.Root>
</header>
