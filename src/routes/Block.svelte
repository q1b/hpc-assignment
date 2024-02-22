<script lang="ts">
	import type { Snippet } from "svelte";

	let { children, active, index,...others } = $props<{
		children: Snippet;
		active?: boolean;
		index: number;
	}>();
</script>

<div aria-current={active} style="transition-delay: {(index+1)*250}ms;" class="p-4 transition-colors bg-slate-500 aria-[current='true']:bg-blue-500 text-white tabular-nums" {...others}>
    {@render children()}
</div>