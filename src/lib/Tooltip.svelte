<script lang="ts">
	export let props;
	export let entities;

	let minerDetails = {
		resource: {
			type: '',
			amount: ''
		}
	};

	let conveyorResources: any = [];

	let minerInventory = 0;

	let producerInputResources: any = [];
	let producerOutputResources: any = [];
	let producerProgress = 0;

	const getProducerDetails = () => {
		producerInputResources =
			entities.find(
				(entity: any) => entity.type === 'producer' && entity.x === props.x && entity.y === props.y
			)?.inputResources || [];

		producerOutputResources =
			entities.find(
				(entity: any) => entity.type === 'producer' && entity.x === props.x && entity.y === props.y
			)?.outputResources || [];

		producerProgress = entities.find(
			(entity: any) => entity.type === 'producer' && entity.x === props.x && entity.y === props.y
		)?.progress;
	};

	const getMinerDetails = () => {
		const resource = entities.find(
			(entity: any) => entity.type === 'resource' && entity.x === props.x && entity.y === props.y
		);
		if (resource) minerDetails.resource = resource.details;
		else {
			minerDetails.resource = {
				type: '',
				amount: ''
			};
		}
	};

	const getConveyorResources = () => {
		conveyorResources =
			entities.find((entity: any) => entity.type === 'conveyor' && entity.id === props.id)
				?.resources || [];
	};

	$: entities ? getConveyorResources() : null;

	$: entities ? getMinerDetails() : null;

	$: entities ? (minerInventory = props.inventory) : null;

	$: entities ? getProducerDetails() : null;
</script>

<div class="tooltip">
	{#if props.type === 'conveyor'}
		<div class="conveyorInfo">
			{#if props.direction === 'up'}
				<div>⬆️</div>
			{/if}
			{#if props.direction === 'down'}
				<div>⬇️</div>
			{/if}
			{#if props.direction === 'left'}
				<div>⬅️</div>
			{/if}
			{#if props.direction === 'right'}
				<div>➡️</div>
			{/if}
			{#if conveyorResources.length === 0}
				<div>No resources</div>
			{/if}
			<div class="resources">
				Count of resources: {conveyorResources.length}
				{#each conveyorResources as resource}
					<div class="conveyorResource">{resource.type}</div>
				{/each}
			</div>
		</div>
	{/if}
	{#if props.type === 'miner'}
		<div class="minerDetails">
			<div>Resource: {minerDetails.resource?.type}</div>
			<div>Amount: {minerDetails.resource?.amount}</div>
			<div>Miner Inventory: {minerInventory}</div>
		</div>
	{/if}
	{#if props.type === 'producer'}
		<div class="producerInfo">
			<div>Producer {props.x}, {props.y}</div>
			<div>Progress: {producerProgress}</div>
			<div>
				Input Resources: {producerInputResources.length}
				{producerInputResources[0]?.type || ''}
			</div>
			<div>
				Output Resources: {producerOutputResources.length}
				{producerOutputResources[0]?.type || ''}
			</div>
		</div>
	{/if}
</div>

<style>
	.tooltip {
		position: absolute;
		background-color: white;
		border: 1px solid black;
		height: 300px;
		width: 300px;
		bottom: 0px;
		right: 0px;
		z-index: 10;
	}

	.resources {
		display: flex;
		flex-wrap: wrap;
		gap: 5px;
	}

	.conveyorResource {
		width: 30px;
		height: 30px;
		border: 1px solid black;
	}
</style>
