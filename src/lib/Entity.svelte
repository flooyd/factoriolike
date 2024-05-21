<script lang="ts">
	import Conveyor from './Conveyor.svelte';
	import Producer from './Producer.svelte';

	export let props;
	export let mouseOver;
	export let mouseLeave;
	export let entities;

	const { x, y, size, id, type } = props;

	const getStyle = () => {
		const baseString = `top: ${y}px; left: ${x}px; width: ${size}px; height: ${size}px;`;
		return baseString;
	};
</script>

<button
	on:mouseenter={mouseOver}
	on:focus={mouseOver}
	on:mouseleave={mouseLeave}
	on:blur={mouseLeave}
	{id}
	class="entity"
	style={getStyle()}
>
	{#if type === 'resource'}
		<div class="resource">{props.details?.type}</div>
	{/if}
	{#if type === 'conveyor'}
		<Conveyor {props} {entities} />
	{/if}
	{#if type === 'producer'}
		<Producer {props} {entities} />
	{/if}
	{#if type === 'miner'}
		<div class="miner">
			{#if props.direction === 'up'}
				<div>^</div>
			{/if}
			{#if props.direction === 'down'}
				<div>v</div>
			{/if}
			{#if props.direction === 'left'}
				<div>&lt;</div>
			{/if}
			{#if props.direction === 'right'}
				<div>&gt;</div>
			{/if}
		</div>
	{/if}
</button>

<style>
	.entity {
		position: absolute;
		background-color: white;
		border: 1px solid black;
	}

	:global(.resource, .conveyor, .miner) {
		width: 100%;
		height: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
		position: relative;
	}

	.resource {
		background-color: grey;
	}

	.miner {
		background-color: green;
		font-weight: bold;
	}
</style>
