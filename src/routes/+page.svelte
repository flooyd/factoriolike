<script>
	// @ts-nocheck
	import { onMount } from 'svelte';
	import Entity from '$lib/Entity.svelte';
	import Tooltip from '$lib/Tooltip.svelte';
	import Inventory from '$lib/Inventory.svelte';

	let cells = [];
	let activeEntity = null;
	let player = {
		posX: 0,
		posY: 3,
		size: 30,
		color: 'blue',
		inventory: []
	};
	let showInventory = false;
	let yStart = 0;
	let yEnd = 0;
	let xStart = 0;
	let xEnd = 0;
	let entities = [
		{
			type: 'resource',
			x: 0,
			y: 0,
			size: 30,
			id: '1',
			details: {
				type: 'Fe',
				amount: 10000
			}
		},
		{
			type: 'resource',
			x: 30,
			y: 0,
			size: 30,
			id: '2',
			details: {
				type: 'Fe',
				amount: 1000
			}
		},
		{
			type: 'resource',
			x: 60,
			y: 0,
			size: 30,
			id: '3',
			details: {
				type: 'Fe',
				amount: 100
			}
		},
		{
			type: 'resource',
			x: 60,
			y: 30,
			size: 30,
			id: '4',
			details: {
				type: 'Fe',
				amount: 100
			}
		},
		{
			type: 'miner',
			x: 0,
			y: 0,
			size: 30,
			id: '17',
			direction: 'down',
			inventory: 0,
			resourceType: 'Fe'
		},
		{
			type: 'miner',
			x: 30,
			y: 0,
			size: 30,
			id: '7',
			direction: 'down',
			inventory: 0,
			resourceType: 'Fe'
		},
		{
			type: 'miner',
			x: 60,
			y: 0,
			size: 30,
			id: '711',
			direction: 'right',
			inventory: 0,
			resourceType: 'Fe'
		},
		{
			type: 'conveyor',
			x: 0,
			y: 30,
			size: 30,
			id: '81',
			direction: 'down',
			resources: []
		},
		{
			type: 'conveyor',
			x: 0,
			y: 60,
			size: 30,
			id: '8',
			direction: 'down',
			resources: []
		},
		{
			type: 'resource',
			x: 240,
			y: 0,
			size: 30,
			id: Math.random().toString(),
			details: {
				type: 'Fe',
				amount: 10000
			}
		},
		{
			type: 'resource',
			x: 180,
			y: 0,
			size: 30,
			id: Math.random().toString(),
			details: {
				type: 'Cu',
				amount: 10000
			}
		},
		{
			type: 'resource',
			x: 210,
			y: 0,
			size: 30,
			id: Math.random().toString(),
			details: {
				type: 'Cu',
				amount: 10000
			}
		},
		{
			type: 'miner',
			x: 210,
			y: 0,
			size: 30,
			id: Math.random().toString(),
			direction: 'down',
			inventory: 0,
			resourceType: 'Cu'
		},
		{
			type: 'miner',
			x: 240,
			y: 0,
			size: 30,
			id: Math.random().toString(),
			direction: 'down',
			inventory: 0,
			resourceType: 'Fe'
		},
		{
			type: 'conveyor',
			x: 240,
			y: 30,
			size: 30,
			id: Math.random().toString(),
			direction: 'down',
			resources: []
		},
		{
			type: 'conveyor',
			x: 30,
			y: 30,
			size: 30,
			id: '91',
			direction: 'down',
			resources: []
		},
		{
			type: 'conveyor',
			x: 30,
			y: 60,
			size: 30,
			id: '123555325',
			direction: 'down',
			resources: []
		},
		{
			type: 'conveyor',
			x: 30,
			y: 90,
			size: 30,
			id: '123525525',
			direction: 'down',
			resources: []
		}
	];

	onMount(() => {
		window.addEventListener('keydown', (e) => {
			movePlayer(e, e.key);
		});
		yEnd = Math.floor(window.innerHeight / 30);
		xEnd = Math.floor(window.innerWidth / 30);
		generateCells();
		if (localStorage.getItem('entities')) {
			entities = JSON.parse(localStorage.getItem('entities'));
		}

		let lastMiningTime = 0;
		let lastOutputTime = 0;
		let lastT1MoveTime = 0;
		let lastSaveTime = 0;

		function gameLoop(time) {
			if (time - lastMiningTime > 25) {
				mine();
				lastMiningTime = time;
			}
			if (time - lastOutputTime > 16) {
				outputToConveyor();
				inputToProducer();
				outputFromProducer();
				produce();
				outputToPlayerInventory();
				lastOutputTime = time;
			}
			if (time - lastT1MoveTime > 1) {
				lastT1MoveTime = time;
				moveDownConveyorResources();
				moveRightConveyorResources();
				moveLeftConveyorResources();
				moveUpConveyorResources();
			}
			if (time - lastSaveTime > 5000) {
				localStorage.setItem('entities', JSON.stringify(entities));
				lastSaveTime = time;
			}
			entities = entities;

			requestAnimationFrame(gameLoop);
		}
		requestAnimationFrame(gameLoop);
	});

	const outputToConveyor = () => {
		const miners = entities.filter((entity) => entity.type === 'miner');
		for (let i = 0; i < miners.length; i++) {
			const miner = miners[i];
			const direction = miner.direction;
			const conveyor = entities.find(
				(entity) =>
					entity.type === 'conveyor' &&
					((direction === 'up' && entity.x === miner.x && entity.y === miner.y - 30) ||
						(direction === 'down' && entity.x === miner.x && entity.y === miner.y + 30) ||
						(direction === 'left' && entity.x === miner.x - 30 && entity.y === miner.y) ||
						(direction === 'right' && entity.x === miner.x + 30 && entity.y === miner.y))
			);
			if (conveyor && miner.inventory > 0 && conveyor.resources.length < 1) {
				miner.inventory = miner.inventory ? miner.inventory - 1 : 0;
				conveyor.resources?.push({
					type: miner.resourceType,
					left: 0,
					top: 0
				});
				entities = entities;
			}
		}
	};

	const inputToProducer = () => {
		const producers = entities.filter((entity) => entity.type === 'producer');
		for (let i = 0; i < producers.length; i++) {
			const producer = producers[i];
			if (producer.inputFrom === 'down') {
				const conveyor = entities.find(
					(entity) =>
						entity.type === 'conveyor' && entity.x === producer.x && entity.y === producer.y + 30
				);
				if (conveyor && conveyor.resources.length > 0 && producer.inputResources.length < 10) {
					producer.inputResources = [...producer.inputResources, ...conveyor.resources];
					conveyor.resources = [];
					entities = entities;
				}
			}
			if (producer.inputFrom === 'up') {
				const conveyor = entities.find(
					(entity) =>
						entity.type === 'conveyor' && entity.x === producer.x && entity.y === producer.y + 30
				);
				if (conveyor && conveyor.resources.length > 0 && producer.inputResources.length < 10) {
					producer.inputResources = conveyor.resources;
					conveyor.resources = [];
					entities = entities;
				}
			}
			if (producer.inputFrom === 'left') {
				const conveyor = entities.find(
					(entity) =>
						entity.type === 'conveyor' && entity.x === producer.x - 30 && entity.y === producer.y
				);
				if (conveyor && conveyor.resources.length > 0 && producer.inputResources.length < 10) {
					producer.inputResources = conveyor.resources;
					conveyor.resources = [];
					entities = entities;
				}
			}
			if (producer.inputFrom === 'right') {
				const conveyor = entities.find(
					(entity) =>
						entity.type === 'conveyor' && entity.x === producer.x + 30 && entity.y === producer.y
				);
				if (conveyor && conveyor.resources.length > 0 && producer.inputResources.length < 10) {
					producer.inputResources = conveyor.resources;
					conveyor.resources = [];
					entities = entities;
				}
			}
		}
	};

	const outputFromProducer = () => {
		const producers = entities.filter((entity) => entity.type === 'producer');
		for (let i = 0; i < producers.length; i++) {
			const producer = producers[i];
			const conveyor = entities.find(
				(entity) =>
					entity.type === 'conveyor' &&
					((producer.outputTo === 'down' &&
						entity.x === producer.x &&
						entity.y === producer.y + 30) ||
						(producer.outputTo === 'up' &&
							entity.x === producer.x &&
							entity.y === producer.y - 30) ||
						(producer.outputTo === 'left' &&
							entity.x === producer.x - 30 &&
							entity.y === producer.y) ||
						(producer.outputTo === 'right' &&
							entity.x === producer.x + 30 &&
							entity.y === producer.y))
			);
			if (conveyor && producer.outputResources.length > 0 && conveyor.resources.length < 1) {
				conveyor.resources.push({
					type: producer.outputResources[0].type,
					left: 0,
					top: 0
				});
				producer.outputResources = producer.outputResources.slice(1);
			}
		}
	};

	const outputToPlayerInventory = () => {
		const playerCell = cells.find((cell) => cell.posX === player.posX && cell.posY === player.posY);
		const conveyor = entities.find((entity) => {
			return (
				entity.type === 'conveyor' &&
				entity.x === playerCell.posX * 30 &&
				entity.y === playerCell.posY * 30
			);
		});
		if (conveyor && conveyor.resources.length > 0) {
			player.inventory.push(conveyor.resources[0]);
			conveyor.resources = conveyor.resources.slice(1);
			player = player;
		}
	};

	const moveDownConveyorResources = () => {
		const conveyors = entities.filter(
			(entity) => entity.type === 'conveyor' && entity.direction === 'down'
		);
		if (conveyors.length === 0) return;
		//find next conveyor
		for (let i = 0; i < conveyors.length; i++) {
			const conveyor = conveyors[i];
			const nextConveyor = entities.find(
				(entity) =>
					entity.type === 'conveyor' && entity.x === conveyor.x && entity.y === conveyor.y + 30
			);
			if (conveyor.resources.length === 0) continue;
			for (let j = 0; j < conveyor.resources.length; j++) {
				const resource = conveyor.resources[j];
				if (
					nextConveyor &&
					nextConveyor.resources.length < 1 &&
					nextConveyor.direction === 'down'
				) {
					resource.top = resource.top + 3;
					if (resource.top > 30) {
						nextConveyor.resources.push({
							type: resource.type,
							left: resource.left,
							top: 0
						});
						conveyor.resources = conveyor.resources.filter((r) => r !== resource);
					}
				}
				if (
					nextConveyor &&
					nextConveyor.resources.length < 1 &&
					nextConveyor.direction === 'right'
				) {
					resource.top = resource.top + 3;
					if (resource.top > 30) {
						nextConveyor.resources.push({
							type: resource.type,
							left: 0,
							top: 0
						});
						conveyor.resources = conveyor.resources.filter((r) => r !== resource);
					}
				}
				if (
					nextConveyor &&
					nextConveyor.resources.length < 1 &&
					nextConveyor.direction === 'left'
				) {
					resource.top = resource.top + 3;
					if (resource.top > 30) {
						nextConveyor.resources.push({
							type: resource.type,
							left: 0,
							top: 0
						});
						conveyor.resources = conveyor.resources.filter((r) => r !== resource);
					}
				}
				if (nextConveyor && nextConveyor.resources.length < 1 && nextConveyor.direction === 'up') {
					resource.top = resource.top + 3;
					if (resource.top > 30) {
						nextConveyor.resources.push({
							type: resource.type,
							left: 0,
							top: 0
						});
						conveyor.resources = conveyor.resources.filter((r) => r !== resource);
					}
				}
			}
		}
		entities = entities;
	};

	const moveRightConveyorResources = () => {
		const conveyors = entities.filter(
			(entity) => entity.type === 'conveyor' && entity.direction === 'right'
		);
		if (conveyors.length === 0) return;
		//find next conveyor
		for (let i = 0; i < conveyors.length; i++) {
			const conveyor = conveyors[i];
			const nextConveyor = entities.find(
				(entity) =>
					entity.type === 'conveyor' && entity.x === conveyor.x + 30 && entity.y === conveyor.y
			);
			if (conveyor.resources.length === 0) continue;
			for (let j = 0; j < conveyor.resources.length; j++) {
				const resource = conveyor.resources[j];
				if (
					nextConveyor &&
					nextConveyor.resources.length < 1 &&
					nextConveyor.direction === 'down'
				) {
					resource.left = resource.left + 3;
					if (resource.left > 30) {
						nextConveyor.resources.push({
							type: resource.type,
							left: 0,
							top: 0
						});
						conveyor.resources = conveyor.resources.filter((r) => r !== resource);
					}
				}
				if (
					nextConveyor &&
					nextConveyor.resources.length < 1 &&
					nextConveyor.direction === 'right'
				) {
					resource.left = resource.left + 3;
					if (resource.left > 30) {
						nextConveyor.resources.push({
							type: resource.type,
							left: 0,
							top: 0
						});
						conveyor.resources = conveyor.resources.filter((r) => r !== resource);
					}
				}
				if (
					nextConveyor &&
					nextConveyor.resources.length < 1 &&
					nextConveyor.direction === 'left'
				) {
					resource.left = resource.left + 3;
					if (resource.left > 30) {
						nextConveyor.resources.push({
							type: resource.type,
							left: -15,
							top: 0
						});
						conveyor.resources = conveyor.resources.filter((r) => r !== resource);
					}
				}
				if (nextConveyor && nextConveyor.resources.length < 1 && nextConveyor.direction === 'up') {
					resource.left = resource.left + 3;
					if (resource.left > 30) {
						nextConveyor.resources.push({
							type: resource.type,
							left: 0,
							top: 0
						});
						conveyor.resources = conveyor.resources.filter((r) => r !== resource);
					}
				}
			}
		}
		entities = entities;
	};

	const moveLeftConveyorResources = () => {
		const conveyors = entities.filter(
			(entity) => entity.type === 'conveyor' && entity.direction === 'left'
		);
		if (conveyors.length === 0) return;
		//find next conveyor
		for (let i = 0; i < conveyors.length; i++) {
			const conveyor = conveyors[i];
			const nextConveyor = entities.find(
				(entity) =>
					entity.type === 'conveyor' && entity.x === conveyor.x - 30 && entity.y === conveyor.y
			);
			if (conveyor.resources.length === 0) continue;
			for (let j = 0; j < conveyor.resources.length; j++) {
				const resource = conveyor.resources[j];
				if (
					nextConveyor &&
					nextConveyor.resources.length < 1 &&
					nextConveyor.direction === 'down'
				) {
					resource.left = resource.left - 3;
					if (resource.left < -30) {
						nextConveyor.resources.push({
							type: resource.type,
							left: 0,
							top: 0
						});
						conveyor.resources = conveyor.resources.filter((r) => r !== resource);
					}
				}
				if (
					nextConveyor &&
					nextConveyor.resources.length < 1 &&
					nextConveyor.direction === 'right'
				) {
					resource.left = resource.left - 3;
					if (resource.left < 0) {
					}
				}
				if (
					nextConveyor &&
					nextConveyor.resources.length < 1 &&
					nextConveyor.direction === 'left'
				) {
					resource.left = resource.left - 3;
					if (resource.left < -30) {
						nextConveyor.resources.push({
							type: resource.type,
							left: 0,
							top: 0
						});
						conveyor.resources = conveyor.resources.filter((r) => r !== resource);
					}
				}
				if (nextConveyor && nextConveyor.resources.length < 1 && nextConveyor.direction === 'up') {
					resource.left = resource.left - 3;
					if (resource.left < -30) {
						nextConveyor.resources.push({
							type: resource.type,
							left: 0,
							top: 0
						});
						conveyor.resources = conveyor.resources.filter((r) => r !== resource);
					}
				}
			}
		}
		entities = entities;
	};

	const moveUpConveyorResources = () => {
		const conveyors = entities.filter(
			(entity) => entity.type === 'conveyor' && entity.direction === 'up'
		);
		if (conveyors.length === 0) return;
		//find next conveyor
		for (let i = 0; i < conveyors.length; i++) {
			const conveyor = conveyors[i];
			const nextConveyor = entities.find(
				(entity) =>
					entity.type === 'conveyor' && entity.x === conveyor.x && entity.y === conveyor.y - 30
			);
			if (conveyor.resources.length === 0) continue;
			for (let j = 0; j < conveyor.resources.length; j++) {
				const resource = conveyor.resources[j];
				if (nextConveyor && nextConveyor.resources.length < 1 && nextConveyor.direction === 'up') {
					resource.top = resource.top - 3;
					if (resource.top < -30) {
						nextConveyor.resources.push({
							type: resource.type,
							left: 0,
							top: 0
						});
						conveyor.resources = conveyor.resources.filter((r) => r !== resource);
					}
				}
				if (
					nextConveyor &&
					nextConveyor.resources.length < 1 &&
					nextConveyor.direction === 'right'
				) {
					resource.top = resource.top - 3;
					if (resource.top < -30) {
						nextConveyor.resources.push({
							type: resource.type,
							left: 0,
							top: 0
						});
						conveyor.resources = conveyor.resources.filter((r) => r !== resource);
					}
				}
				if (
					nextConveyor &&
					nextConveyor.resources.length < 1 &&
					nextConveyor.direction === 'left'
				) {
					resource.top = resource.top - 3;
					if (resource.top < -30) {
						nextConveyor.resources.push({
							type: resource.type,
							left: 0,
							top: 0
						});
						conveyor.resources = conveyor.resources.filter((r) => r !== resource);
					}
				}
				if (nextConveyor && nextConveyor.resources.length < 1 && nextConveyor.direction === 'up') {
					resource.top = resource.top - 3;
					if (resource.top < -30) {
						nextConveyor.resources.push({
							type: resource.type,
							left: 0,
							top: 0
						});
						conveyor.resources = conveyor.resources.filter((r) => r !== resource);
					}
				}
			}
		}
		entities = entities;
	};

	const mine = () => {
		const miners = entities.filter((entity) => entity.type === 'miner');
		for (let i = 0; i < miners.length; i++) {
			const miner = miners[i];
			const resource = entities.find(
				(entity) => entity.type === 'resource' && entity.x === miner.x && entity.y === miner.y
			);
			if (resource && resource.details && (miner.inventory ?? 0) < 100) {
				resource.details.amount = resource.details.amount - 3;
				miner.inventory = (miner.inventory ?? 0) + 3;
				if (resource.details.amount <= 0) {
					entities = entities.filter((entity) => entity.id !== resource.id);
				}
				entities = entities;
			}
		}
	};

	const produce = () => {
		const producers = entities.filter((entity) => entity.type === 'producer');
		for (let i = 0; i < producers.length; i++) {
			const producer = producers[i];
			if (producer.inputResources.length > 0) {
				producer.progress = producer.progress + 3;
				if (producer.progress >= 100 && producer.outputResources.length < 10) {
					producer.progress = 0;
					let type = '';
					if (producer.inputResources[0].type === 'Fe') {
						type = 'Iron Plate';
					}
					if (producer.inputResources[0].type === 'Cu') {
						type = 'Copper Plate';
					}
					producer.inputResources.pop();
					producer.outputResources.push({
						type
					});
				}
				if (producer.outputResources.length === 10) {
					producer.progress = 0;
				}
			}
		}
		entities = entities;
	};

	const generateCells = () => {
		cells = [];
		for (let i = yStart; i < yEnd; i++) {
			for (let j = xStart; j < xEnd; j++) {
				cells.push({
					posX: j,
					posY: i,
					size: 30,
					color: 'white'
				});
				cells = cells;
			}
		}
	};

	const movePlayer = (e, moveDirection) => {
		e.preventDefault();
		if (moveDirection === 'w') {
			if (player.posY === yStart - 1) return;
			player.posY = player.posY - 1;
		}
		if (moveDirection === 's') {
			if (player.posY === yEnd) return;
			player.posY = player.posY + 1;
		}
		if (moveDirection === 'a') {
			if (player.posX === xStart - 1) return;
			player.posX = player.posX - 1;
		}
		if (moveDirection === 'd') {
			if (player.posX === xEnd) return;
			player.posX = player.posX + 1;
		}
		if (moveDirection === 'ArrowDown') {
			const entity = entities.find(
				(entity) =>
					entity.type === 'conveyor' &&
					entity.x === player.posX * 30 &&
					entity.y === player.posY * 30
			);
			if (entity) {
				entity.direction = 'down';
			} else {
				entities.push({
					type: 'conveyor',
					x: player.posX * 30,
					y: player.posY * 30,
					size: 30,
					id: Math.random().toString(),
					direction: 'down',
					resources: []
				});
			}
		}
		if (moveDirection === 'ArrowRight') {
			const entity = entities.find(
				(entity) =>
					entity.type === 'conveyor' &&
					entity.x === player.posX * 30 &&
					entity.y === player.posY * 30
			);
			if (entity) {
				entity.direction = 'right';
			} else {
				entities.push({
					type: 'conveyor',
					x: player.posX * 30,
					y: player.posY * 30,
					size: 30,
					id: Math.random().toString(),
					direction: 'right',
					resources: []
				});
			}
		}
		if (moveDirection === 'ArrowUp') {
			const entity = entities.find(
				(entity) =>
					entity.type === 'conveyor' &&
					entity.x === player.posX * 30 &&
					entity.y === player.posY * 30
			);
			if (entity) {
				entity.direction = 'up';
			} else {
				entities.push({
					type: 'conveyor',
					x: player.posX * 30,
					y: player.posY * 30,
					size: 30,
					id: Math.random().toString(),
					direction: 'up',
					resources: []
				});
			}
		}
		if (moveDirection === 'ArrowLeft') {
			const entity = entities.find(
				(entity) =>
					entity.type === 'conveyor' &&
					entity.x === player.posX * 30 &&
					entity.y === player.posY * 30
			);
			if (entity) {
				entity.direction = 'left';
			} else {
				entities.push({
					type: 'conveyor',
					x: player.posX * 30,
					y: player.posY * 30,
					size: 30,
					id: Math.random().toString(),
					direction: 'left',
					resources: []
				});
			}
		}
		if (moveDirection === 'Delete') {
			entities = entities.filter(
				(entity) => entity.x !== player.posX * 30 || entity.y !== player.posY * 30
			);
		}
		if (moveDirection === 'c') {
			entities = entities.filter((entity) => entity.type !== 'conveyor');
		}
		if (moveDirection === 'x') {
			entities = entities.map((entity) => {
				if (entity.type === 'conveyor') {
					entity.resources = [];
				}
				return entity;
			});
		}
		if (moveDirection === 'p') {
			entities.push({
				type: 'producer',
				x: player.posX * 30,
				y: player.posY * 30,
				size: 30,
				id: Math.random().toString(),
				inputFrom: 'down',
				outputTo: 'up',
				inputResources: [],
				outputResources: [],
				progress: 0
			});
		}
		if (moveDirection === 'i') {
			showInventory = !showInventory;
		}

		player = player;
	};

	// @ts-ignore
	const handleMouseOver = (entity) => {
		activeEntity = entity;
	};

	const handleMouseLeave = () => {
		activeEntity = null;
	};
</script>

{#each cells as cell ('x' + cell.posX + 'y' + cell.posY)}
	<div
		class="cell"
		style="height: {cell.size}px;
        width: {cell.size}px;
        background-color: {cell.color};
        position: absolute;
        top: {cell.posY * cell.size}px;
        left: {cell.posX * cell.size}px;
      "
	>
		<div class="coords">{cell.posX} {cell.posY}</div>
	</div>
{/each}
{#each entities as entity (entity.id)}
	<Entity
		props={entity}
		mouseOver={() => handleMouseOver(entity)}
		mouseLeave={() => handleMouseLeave()}
		{entities}
	/>
{/each}
<div
	class="cell player"
	style="height: {player.size}px;
        width: {player.size}px;
        background-color: {'red'};
        position: absolute;
        top: {player.posY * 30}px;
        left: {player.posX * 30}px;
      "
></div>
{#if activeEntity}
	<Tooltip props={activeEntity} {entities} />
{/if}
{#if showInventory}
	<Inventory inventory={player.inventory} />
{/if}

<style>
	:global(*) {
		box-sizing: border-box;
	}
	:global(button) {
		padding: 0px;
	}
	.cell {
		border: 1px solid black;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.cell.player {
		background: red;
		z-index: 2;
	}
	.coords {
		position: absolute;
		bottom: 0px;
		font-size: 10px;
	}
</style>
