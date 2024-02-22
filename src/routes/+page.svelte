<script lang="ts">
	import { get } from 'svelte/store';
	import Block from './Block.svelte';
	import split from 'just-split';

	let number_of_processors = $state(8);
	let selected_processor = $state(2);
	let selected_memory = $state(7);

	function nextAddress(i: number) {
		if (i >= 0 && i < number_of_processors / 2) {
			return 2 * i;
		} else {
			return 2 * i + 1 - number_of_processors;
		}
	}

	function getBinary(decimal: number) {
		let binary = '';
		while (decimal > 0) {
			if (decimal % 2 == 1) {
				binary = '1' + binary;
			} else {
				binary = '0' + binary;
			}
			// divide number by 2.
			decimal = Math.floor(decimal / 2);
		}
		return binary.toString().padStart(number_of_stages, '0');
	}

	function isBitSignificant(stage: number) {
		return getBinary(selected_processor)[stage] === getBinary(selected_memory)[stage];
	}

	const number_of_stages = $derived(Math.log2(number_of_processors));

	const processors: {
		value: number;
		binary: string;
	}[] = $derived(
		Array.from({ length: number_of_processors }, (_, i) => ({
			value: i,
			binary: getBinary(i),
		}))
	);

	const getStages = (number_of_stages: number) => {
		return Array.from({ length: number_of_stages }, (_, i) => ({
			switches: split(
				Array.from({ length: number_of_processors }, (_, i) => ({
					value: i,
					binary: getBinary(i),
					active: false
				})),
				2
			) as [
				{
					value: number;
					binary: string;
					active?: boolean;
				},
				{
					value: number;
					binary: string;
					active?: boolean;
				}
			][]
		}));
	};
	let stages: {
		switches: [
			{
				value: number;
				binary: string;
				active?: boolean;
			},
			{
				value: number;
				binary: string;
				active?: boolean;
			}
		][];
	}[] = $state(getStages(number_of_stages));
	
	$effect(() => {
		stages = getStages(number_of_stages);
	});

	$effect(() => {
		let currentProcessor = selected_processor;
		let previousProcessor = currentProcessor;
		console.log('selected processor', currentProcessor, getBinary(currentProcessor));
		for (let stageIndex = 0; stageIndex < stages.length; stageIndex++) {
			previousProcessor = currentProcessor;
			currentProcessor = nextAddress(currentProcessor);
			console.log('Points to ', currentProcessor);
			const stage = stages[stageIndex];
			let z: number | null = null;
			const switchIndex = stage.switches.findIndex((Switch) => {
				if (Switch[0].value === currentProcessor) {
					z = 0;
					return true;
				} else if (Switch[1].value === currentProcessor) {
					z = 1;
					return true;
				}
				return false;
			});

			if (!isBitSignificant(stageIndex) && switchIndex !== -1) {
				z = z === 0 ? 1 : 0;
				currentProcessor = stage.switches[switchIndex][z].value;
				stages[stageIndex].switches[switchIndex][z].active = true;
				// if (currentProcessor % 2 === 0) {
				// 	currentProcessor = currentProcessor + 1;
				// } else {
				// 	currentProcessor = currentProcessor - 1;
				// }
				console.log('APPLIED CROSSOVER', currentProcessor, getBinary(currentProcessor));
			} else {
				stages[stageIndex].switches[switchIndex][z].active = true;
				console.log('APPLIED PASSOVER', currentProcessor, getBinary(currentProcessor));
			}
		}
		return () => stages = getStages(number_of_stages);
	});
</script>

<header class="w-full max-w-5xl p-4">High Performance Computing Assignment</header>

<main class="w-full max-w-5xl p-4">
	<h1 class="mb-4">
		Processor Count: <select bind:value={number_of_processors}>
			<option value={2}>2</option>
			<option value={4}>4</option>
			<option value={8}>8</option>
			<option value={16}>16</option>
		</select>
	</h1>
	<div class="flex flex-col gap-y-2 mb-6">
	<h4>Processor: <select bind:value={selected_processor}>
		{#each Array.from({length: number_of_processors}) as _,i}
			<option value={i}>{i}</option>
		{/each}
	</select></h4>
	<h4>Memory: <select bind:value={selected_memory}>
		{#each Array.from({length: number_of_processors}) as _,i}
			<option value={i}>{i}</option>
		{/each}
	</select></h4>
	</div>
	<div class="flex items-center gap-x-6">
		<div class="flex flex-col gap-y-4">
			{#each processors as processor}
				<Block index={0} active={selected_processor === processor.value} >
					{processor.binary}
				</Block>
			{/each}
		</div>
		{#each stages as stage, i}
			<div class="flex flex-col gap-y-4 divide-y-2">
				{#each stage.switches as Switch}
					<div class="">
						{#each Switch as item}
							<Block index={i} active={item.active}>{item.binary}</Block>
						{/each}
					</div>
				{/each}
			</div>
		{/each}
		<div class="flex flex-col gap-y-4">
			{#each processors as processor}
				<Block active={selected_memory === processor.value} 
					>{processor.binary}</Block
				>
			{/each}
		</div>
	</div>
</main>
