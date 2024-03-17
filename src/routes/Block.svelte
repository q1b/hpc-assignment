<script lang="ts">
	import type { Snippet } from "svelte";

	let { children, active, block, index,el,...others } = $props<{
		children: Snippet;
		active?: boolean;
		index: number;
		block?: 'before' | 'after';
		el: HTMLElement | null;
	}>();
	const indicator = {
		'after' : "aria-[current='true']:bg-blue-500",
		'before' : "aria-[current='true']:bg-pink-600",
		undefined : "aria-[current='true']:bg-blue-500"
	}[block ?? 'undefined']
</script>

<div bind:this={el} aria-current={active} style="transition-delay: {(index+1)*250}ms;" class="p-4 transition-colors bg-slate-500 {indicator} text-white tabular-nums rounded-md m-px" {...others}>
    {@render children()}
</div>