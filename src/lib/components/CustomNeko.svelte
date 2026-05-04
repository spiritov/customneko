<script lang="ts">
	type Animation = 'N' | 'E' | 'S' | 'W' | 'NE' | 'SE' | 'SW' | 'NW' | 'Sleep';

	type SpriteIndex = {
		row: number;
		length: number;
	};

	type Props = {
		image_path: string;
		sprite_width: number;
		sprite_height: number;
		update_ms?: number;
		sleep_time?: number;
		sprite_move_speed?: number;
	};

	// width, height: each spritesheet frame is expected to have the same width and height
	// update_ms: how fast sprite animations should cycle frames
	// sleep time (breathing) ms is update_ms * sleep_time
	// move_speed: sprite move speed in pixels, every update_ms ms
	let {
		image_path,
		sprite_width,
		sprite_height,
		update_ms = 75,
		sleep_time = 30,
		sprite_move_speed = 10
	}: Props = $props();

	// set: each row the spritesheet to an Animation's entry
	const spriteIndexes = new Map<Animation, SpriteIndex>([
		['Sleep', { row: 0, length: 2 }],
		['S', { row: 1, length: 7 }],
		['SE', { row: 2, length: 7 }],
		['E', { row: 3, length: 7 }],
		['NE', { row: 4, length: 7 }],
		['N', { row: 5, length: 7 }],
		['NW', { row: 6, length: 7 }],
		['W', { row: 7, length: 7 }],
		['SW', { row: 8, length: 7 }]
	]);

	const sprite_width_offset = $derived(sprite_width / 1.5);
	const sprite_height_offset = $derived(sprite_height / 1.5);
	const sprite_sleep_distance = $derived(
		Math.sqrt(sprite_width_offset ** 2 + sprite_height_offset ** 2)
	);

	let allowUpdates = false;
	let lastTimestamp = 0;

	let mouseX = 0;
	let mouseY = 0;

	// sleep by default
	let spriteAnimation: SpriteIndex = { row: 0, length: 2 };
	let sleepTimer = 0;

	// reactive variables used to set the sprite's position and frame
	const spriteIndex = $state({
		row: 0,
		frame: 0
	});
	let spriteX = $state(80);
	let spriteY = $state(80);

	function onWindowAnimationFrame(timestamp: number) {
		if (timestamp - lastTimestamp > update_ms && allowUpdates) {
			lastTimestamp = timestamp;
			update();
		}
		window.requestAnimationFrame(onWindowAnimationFrame);
	}

	function setSpriteDirection(x: number, y: number) {
		// NE, E, SE
		if (x > 0.5) {
			if (y > 0.5) {
				spriteAnimation = spriteIndexes.get('SE') as SpriteIndex;
			} else if (y < -0.5) {
				spriteAnimation = spriteIndexes.get('NE') as SpriteIndex;
			} else {
				spriteAnimation = spriteIndexes.get('E') as SpriteIndex;
			}
		}
		// NW, W, SW
		else if (x < -0.5) {
			if (y > 0.5) {
				spriteAnimation = spriteIndexes.get('SW') as SpriteIndex;
			} else if (y < -0.5) {
				spriteAnimation = spriteIndexes.get('NW') as SpriteIndex;
			} else {
				spriteAnimation = spriteIndexes.get('W') as SpriteIndex;
			}
		}
		// N, S
		else {
			if (y > 0) {
				spriteAnimation = spriteIndexes.get('S') as SpriteIndex;
			} else {
				spriteAnimation = spriteIndexes.get('N') as SpriteIndex;
			}
		}
	}

	// update loop
	function update() {
		const diffX = mouseX - spriteX - sprite_width_offset;
		const diffY = mouseY - spriteY - sprite_height_offset;

		const distance = Math.sqrt(diffX ** 2 + diffY ** 2);
		const normalX = diffX / distance;
		const normalY = diffY / distance;

		// sleep
		if (distance < sprite_sleep_distance) {
			spriteAnimation = spriteIndexes.get('Sleep') as SpriteIndex;
			sleepTimer = (sleepTimer + 1) % sleep_time;
			spriteIndex.frame = sleepTimer < sleep_time / 2 ? 0 : 1;
		}
		// moving
		else {
			setSpriteDirection(normalX, normalY);
			spriteX += normalX * sprite_move_speed;
			spriteY += normalY * sprite_move_speed;
		}

		spriteIndex.row = spriteAnimation.row;
		spriteIndex.frame = (spriteIndex.frame + 1) % spriteAnimation.length;
	}

	$effect(() => {
		// mdn: requestAnimationFrame call frequency generally matches display refresh rate
		const frame = window.requestAnimationFrame(onWindowAnimationFrame);

		return () => {
			window.cancelAnimationFrame(frame);
		};
	});
</script>

<svelte:window
	onmousemove={(m: MouseEvent) => {
		allowUpdates = true;
		mouseX = m.clientX;
		mouseY = m.clientY;
	}}
	onmouseout={() => {
		allowUpdates = false;
	}}
/>

<div
	style:image-rendering="pixelated"
	style:background="url({image_path})"
	style:background-position="{spriteIndex.frame * sprite_width * -1}px {spriteIndex.row *
		sprite_height *
		-1}px"
	style:left="{spriteX}px"
	style:top="{spriteY}px"
	style:height="{sprite_height}px"
	style:width="{sprite_width}px"
	style:transform="scale(2, 2)"
	style:background-repeat="no-repeat"
	style:position="fixed"
	style:z-index="9999"
	style:pointer-events="none"
></div>
