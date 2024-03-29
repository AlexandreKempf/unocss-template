<script>
	import P5 from 'p5-svelte';
	import * as math from 'mathjs';

	export let levelName;
	export let playerName;

	let background;
	let foreground;
	let walkableMap;
	let player;

	let width = 500;
	let height = 500;
	let FPS = 15;

	let position = [1080, 1220];
	let speed = 8;
	let animationInformation = ['idle', 'down', 0];
	let animationTable = { idle: 0, walk: 1152, right: 0, up: 288, left: 576, down: 864 };

	function local_display(
		p5,
		background,
		foreground,
		player,
		position,
		oldPosition,
		animationInformation
	) {
		p5.background('white');
		p5.image(
			background,
			0,
			0,
			width,
			height,
			math.ceil(position[0] - width / 2),
			math.ceil(position[1] - height / 2),
			width,
			height
		);
		animationInformation[0] =
			vectorNorm(math.subtract(position, oldPosition)) == 0 ? 'idle' : 'walk';

		let displacementVector = math.subtract(position, oldPosition);
		if (displacementVector[0] > 0) animationInformation[1] = 'right';
		if (displacementVector[0] < 0) animationInformation[1] = 'left';
		if (displacementVector[1] > 0) animationInformation[1] = 'down';
		if (displacementVector[1] < 0) animationInformation[1] = 'up';

		animationInformation[2] = (animationInformation[2] + 1) % 6;

		let animationScore =
			animationTable[animationInformation[0]] +
			animationTable[animationInformation[1]] +
			48 * animationInformation[2];

		p5.image(
			player,
			math.ceil(width / 2),
			math.ceil(height / 2),
			48,
			96,
			animationScore,
			0,
			48,
			96
		);
		p5.image(
			foreground,
			0,
			0,
			width,
			height,
			math.ceil(position[0] - width / 2),
			math.ceil(position[1] - height / 2),
			width,
			height
		);
	}

	function vectorNorm(vector) {
		return math.sqrt(math.sum(math.map(vector, math.square)));
	}

	function handleWalls(walkableMap, player, position, derivedPosition) {
		let keypoints = [
			math.add(position, [10, 96 - 28]),
			math.add(position, [48 - 10, 96 - 28]),
			math.add(position, [48 - 10, 96 - 2]),
			math.add(position, [10, 96 - 2])
		];

		let stopHorizontal = math
			.add(keypoints, [derivedPosition[0], 0])
			.map((pos) => (pos[0] + pos[1] * walkableMap.width) * 4)
			.some((index) => walkableMap.pixels[index] == 0);
		let stopVertical = math
			.add(keypoints, [0, derivedPosition[1]])
			.map((pos) => (pos[0] + pos[1] * walkableMap.width) * 4)
			.some((index) => walkableMap.pixels[index] == 0);
		let stopBoth = math
			.add(keypoints, derivedPosition)
			.map((pos) => (pos[0] + pos[1] * walkableMap.width) * 4)
			.some((index) => walkableMap.pixels[index] == 0);

		if (stopHorizontal) derivedPosition[0] = 0;
		if (stopVertical) derivedPosition[1] = 0;
		if (stopBoth && !stopVertical) derivedPosition[0] = 0;
		if (stopBoth && !stopHorizontal) derivedPosition[1] = 0;
		return derivedPosition;
	}

	const sketch = (p5) => {
		p5.preload = () => {
			background = p5.loadImage(`levels/${levelName}/background.webp`);
			foreground = p5.loadImage(`levels/${levelName}/foreground.webp`);
			player = p5.loadImage(`players/${playerName}.webp`);
			walkableMap = p5.loadImage(`levels/${levelName}/walkable.webp`);
		};

		p5.setup = () => {
			walkableMap.loadPixels();
			p5.createCanvas(width, height);
			// p5.image(background, 0, 0);
			p5.image(player, position[0], position[1], 48, 96, 0, 0, 48, 96);
			// p5.image(foreground, 0, 0);
			p5.frameRate(FPS);
		};

		p5.draw = () => {
			let oldPosition = position;

			// displacement
			let derivedPosition = [0, 0];
			if (p5.keyIsDown(p5.LEFT_ARROW)) derivedPosition[0] -= 1;
			if (p5.keyIsDown(p5.RIGHT_ARROW)) derivedPosition[0] += 1;
			if (p5.keyIsDown(p5.UP_ARROW)) derivedPosition[1] -= 1;
			if (p5.keyIsDown(p5.DOWN_ARROW)) derivedPosition[1] += 1;
			let derivedPositionNorm = vectorNorm(derivedPosition);
			if (derivedPositionNorm != 0) {
				derivedPosition = math.multiply(math.divide(derivedPosition, derivedPositionNorm), speed);
				derivedPosition = math.round(derivedPosition);
				derivedPosition = handleWalls(walkableMap, player, position, derivedPosition);
				position = math.add(position, derivedPosition);
			}

			// display
			local_display(
				p5,
				background,
				foreground,
				player,
				position,
				oldPosition,
				animationInformation
			);
		};
	};
</script>

<div class="m-0 w-full h-full">
	<P5 {sketch} />
</div>
