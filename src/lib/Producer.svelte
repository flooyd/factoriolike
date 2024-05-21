<script>
	export let props;
	export let entities;

	const getStyle = () => {
		const progress = props.progress;
		return `background: linear-gradient(to right, green ${progress}%, lightgrey ${progress}%);`;
	};

	let style = getStyle();

	$: props ? (style = getStyle()) : null;
</script>

<div class="producer" {style}>
	<div class="outputArrow">^</div>
	<div class="title">Prod</div>
	<div class="inputArrow">^</div>
</div>

<style>
	.producer {
		height: 100%;
		display: flex;
		position: relative;
		flex-direction: column;
	}

	.outputArrow,
	.inputArrow {
		font-size: 10px;
		font-weight: bold;
	}

	.title {
		font-size: 10px;
		font-weight: bold;
	}
</style>
