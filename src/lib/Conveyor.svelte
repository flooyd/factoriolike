<script lang="ts">
	export let props;
	export let entities: any;
	export let resources: any = [];

	$: entities
		? (resources = entities.find((entity: any) => entity.id === props.id).resources)
		: null;

	const getResourceStyle = (resource: any) => {
		const baseString = `top: ${resource.top}px; left: ${resource.left}px;`;
		if (resource.type === 'Fe') return `${baseString} background-color: grey; color: white;`;
		if (resource.type === 'Cu') return `${baseString} background-color: orange; color: white;`;
		if (resource.type === 'Copper Plate')
			return `${baseString} background-color: gold; color: black;`;
		if (resource.type === 'Iron Plate')
			return `${baseString} background-color: silver; color: black;`;
	};

	const getResourceType = (resource: { type: string }) => {
		if (resource.type === 'Fe') return 'Fe';
		if (resource.type === 'Cu') return 'Cu';
		if (resource.type === 'Iron Plate') return 'IP';
		if (resource.type === 'Copper Plate') return 'CP';
	};
</script>

<div class="conveyor">
	{#if props.direction === 'up'}
		<div class="arrow">^</div>
	{/if}
	{#if props.direction === 'down'}
		<div class="arrow">v</div>
	{/if}
	{#if props.direction === 'left'}
		<div class="arrow">&lt;</div>
	{/if}
	{#if props.direction === 'right'}
		<div class="arrow">&gt;</div>
	{/if}
	{#each resources as resource}
		<div class="resource" style={getResourceStyle(resource)}>{getResourceType(resource)}</div>
	{/each}
</div>

<style>
	.conveyor {
		background-color: blue;
		height: 100%;
		display: flex;
		position: relative;
	}

	.arrow {
		font-size: 25px;
		position: absolute;
	}

	.resource {
		width: 15px;
		height: 15px;
		background: white;
		position: absolute;
		z-index: 2;
		box-shadow: 3px 0px 5px 0px rgba(0, 0, 0, 1);
	}
</style>
