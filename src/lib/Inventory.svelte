<script lang="ts">
	export let inventory;
	let uniqueItemTypes: any[] = [];
	const getUniqueItemTypes = () => {
		uniqueItemTypes = [...new Set(inventory.map((item: { type: any }) => item.type))];
	};

	const getInventoryAmount = (type: any) => {
		return inventory.filter((item: { type: any }) => item.type === type).length;
	};

	$: inventory ? getUniqueItemTypes() : null;
</script>

<div class="inventory">
	<div class="title">Inventory</div>
	<div class="items">
		{#each uniqueItemTypes as type}
			<div class="item">
				{type} - {getInventoryAmount(type)}
			</div>
		{/each}
	</div>
</div>

<style>
	.inventory {
		position: absolute;
		right: 0px;
		top: 0px;
		width: 300px;
		height: 100%;
		background: white;
		padding: 10px;
	}

	.title {
		font-size: 20px;
		font-weight: bold;
		margin-bottom: 20px;
	}

	.items {
		display: flex;
		flex-wrap: wrap;
		gap: 5px;
		flex-direction: column;
	}
</style>
