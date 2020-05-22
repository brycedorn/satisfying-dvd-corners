<script>
	import { onMount, onDestroy } from 'svelte';

	import DVDLogo from './svg/dvd-logo.svg';
	import gitlab from './svg/gitlab.svg';
	import github from './svg/github.svg';
	import dev from './svg/dev.to.svg';

	let repoName = 'satisfying-dvd-corners';
	let articleLink = 'https://dev.to/bryce/beautiful-dvd-corners-2l1d-temp-slug-4306163';

	let showControls;
	let gridEnabled = true;
	let arrowRotation = 0;
	$: arrowRotationDeg = `${arrowRotation}deg`;
	$: controlOffset = showControls ? '0px' : '60px';
	let dvdBackgroundEnabled = true;
	$: dvdBackground = dvdBackgroundEnabled ? "#beeeef" : "none";

	/* via https://www.heropatterns.com/ */
	$: backgroundImage = gridEnabled ? `url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='100' height='100' viewBox='0 0 100 100'%3E%3Cg fill-rule='evenodd'%3E%3Cg fill='%23000' fill-opacity='0.1'%3E%3Cpath opacity='.5' d='M96 95h4v1h-4v4h-1v-4h-9v4h-1v-4h-9v4h-1v-4h-9v4h-1v-4h-9v4h-1v-4h-9v4h-1v-4h-9v4h-1v-4h-9v4h-1v-4h-9v4h-1v-4H0v-1h15v-9H0v-1h15v-9H0v-1h15v-9H0v-1h15v-9H0v-1h15v-9H0v-1h15v-9H0v-1h15v-9H0v-1h15v-9H0v-1h15V0h1v15h9V0h1v15h9V0h1v15h9V0h1v15h9V0h1v15h9V0h1v15h9V0h1v15h9V0h1v15h9V0h1v15h4v1h-4v9h4v1h-4v9h4v1h-4v9h4v1h-4v9h4v1h-4v9h4v1h-4v9h4v1h-4v9h4v1h-4v9zm-1 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-9-10h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm9-10v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-9-10h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm9-10v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-9-10h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm9-10v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-10 0v-9h-9v9h9zm-9-10h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9zm10 0h9v-9h-9v9z'/%3E%3Cpath d='M6 5V0H5v5H0v1h5v94h1V6h94V5H6z'/%3E%3C/g%3E%3C/g%3E%3C/svg%3E")` : "none";

	let logoWidth = 512;
	let logoHeight = 260;
	let left = 0;
	let top = 0;
	$: leftPx = `${left}px`;
	$: topPx = `${top}px`;

	let looper;
	let cornerImminent;
	let isTopLeftCornerBounce;
	let postCornerBounce;

	let speed = 1;
	let maxSpeed = 1;
	let direction = {
		x: 1,
		y: 1
	};

	let cornerApproachDistance = 100;
	let zoomStopDistance = 20;
	$: approachDeceleration = maxSpeed/(cornerApproachDistance * 1.98);
	let zoomSpeed = 10;
	let zoom = 100;
	$: zoomPercent = `${zoom}%`;

	onMount(async () => {
		cornerImminent = willHitCornerNext(direction);
		isTopLeftCornerBounce = cornerImminent && direction.x === -1 && direction.y === -1;
		if (window.innerWidth <= 640) {
			logoWidth = 256;
			logoHeight = 130;
		}
		alignForCornerHit();
		startMoving();
	});

	function handleInputChange({ target: { value }}) {
		speed = value;
		maxSpeed = value;
	}

	function alignForCornerHit() {
		const maxDistance = Math.min(window.innerWidth - logoWidth, window.innerHeight - logoHeight);
		top = maxDistance;
		left = maxDistance;
		direction = {
			x: -1,
			y: -1
		};
		cornerImminent = true;
		isTopLeftCornerBounce = true;
	}

	function toggleControls() {
		showControls = !showControls;
		arrowRotation += 180;
	}

	function willHitCornerNext() {
		if (direction.x > 0) {
			if (direction.y > 0) { // bottomRight
				return top + (window.innerWidth - left - logoWidth) === window.innerHeight - logoHeight;
			} else { // topRight
				return top - (window.innerWidth - left - logoWidth) === 0;
			}
		} else {
			if (direction.y > 0) { // bottomLeft
				return top + left === window.innerHeight - logoHeight;
			} else { // topLeft
				return top - left === 0;
			}
		}
	};

	function move() {
		let { x, y } = direction;

		let newLeft = left + (x * speed);
		const hitX = newLeft > (window.innerWidth - logoWidth) || newLeft < 0;

		let newTop = top + (y * speed);
		const hitY = newTop > (window.innerHeight - logoHeight) || newTop < 0;

		// bounce
		if (hitX && hitY) {
			x *= -1;
			y *= -1;

			postCornerBounce = true;

			newLeft = left + (x * speed);
			newTop = top + (y * speed);
		} else if (hitX) {
			x *= -1;
			newLeft = left + (x * speed);
		} else if (hitY) {
			y *= -1;
			newTop = top + (y * speed);
		}

		if (hitX || hitY) {
			direction = { x, y };
			cornerImminent = willHitCornerNext({ x, y });
		}

		if (postCornerBounce) {
			if (speed < maxSpeed) {
				speed += approachDeceleration;
			} else {
				speed = maxSpeed;
			}

			if (isTopLeftCornerBounce) {
				const withinZoomStopThreshold = top < zoomStopDistance;

				if (!withinZoomStopThreshold) {
					if (zoom > 100) {
						zoom = Math.max(zoom - zoomSpeed, 100)
					} else {
						zoom = 100;
						postCornerBounce = false;
						isTopLeftCornerBounce = cornerImminent && x === -1 && y === -1;
					}
				}
			}
		} else if (cornerImminent) {
			const withinCornerThreshold = (y > 0 && (window.innerHeight - top - logoHeight) < cornerApproachDistance || (y < 0 && top < cornerApproachDistance));

			if (withinCornerThreshold) {
				speed -= approachDeceleration;

				if (isTopLeftCornerBounce) {
					const withinZoomStopThreshold = (y > 0 && (window.innerHeight - top - logoHeight) < zoomStopDistance || (y < 0 && top < zoomStopDistance));

					if (!withinZoomStopThreshold) {
						zoom += zoomSpeed;
					}
				}
			}
		}

		left = newLeft;
		top = newTop;
	}

	function startMoving() {
		looper = window.requestAnimationFrame(startMoving);
		move();
	}

	function stopMoving() {
    window.cancelAnimationFrame(looper)
	}

	onDestroy(stopMoving);
</script>

<div id="background" style="zoom: {zoomPercent}; background-image: {backgroundImage}">
	<div id="DVD" style="left: {leftPx}; top: {topPx}; background: {dvdBackground}">
		<img src={DVDLogo} />
	</div>
</div>

<div id="controls" style="transform: translateY({ controlOffset })">
	<button id="controls-toggle" on:click={toggleControls}>
		<span style="transform: rotate({ arrowRotationDeg })">⬆️</span>
	</button>
	<div id="controls-inner">
		<label for="speed">
			<span>Speed:</span>
			<input name="speed" type="range" step="1" min="1" max="10" value={speed} on:change={handleInputChange}>
		</label>
		<label for="grid">
			<span>Grid:</span>
			<input name="grid" type="checkbox" bind:checked={gridEnabled}>
		</label>
		<label for="dvd-bg">
			<span>Background:</span>
			<input name="dvd-bg" type="checkbox" bind:checked={dvdBackgroundEnabled}>
		</label>
		<button name="corner" on:click={alignForCornerHit}>Hit Corner Next</button>
		<a href="{articleLink}" class="repo-link">
			<img src={dev} />
		</a>
		<a href="https://github.com/brycedorn/{repoName}" class="repo-link">
			<img src={github} />
		</a>
		<a href="https://gitlab.com/brycedorn/{repoName}" class="repo-link">
			<img src={gitlab} />
		</a>
	</div>
</div>

<style>
	:global(body) {
		margin: 0;
		padding: 0;
		overflow: hidden;
		position: relative;
		font-family: Arial, Helvetica, sans-serif;
	}

	#controls {
		position: fixed;
		z-index: 1;
		bottom: 0;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		width: 100%;
		transition: transform 0.5s;
	}

	#controls-inner {
		width: 600px;
		height: 40px;
		display: flex;
		align-items: center;
		justify-content: space-evenly;
		padding: 10px;
	}

	#controls-toggle {
		cursor: pointer;
		height: 40px;
		width: 60px;
		display: flex;
		align-items: center;
		justify-content: center;
		margin: 0;
		border: none;
		font-size: 16px;
	}

	#controls-toggle:focus {
		outline: none;
	}

	input[type="range"] {
		width: 80px;
	}

	button, input {
		cursor: pointer;
		margin: 0 6px;
	}

	label {
		display: flex;
		align-items: baseline;
	}

	#controls-toggle span {
		width: 16px;
		transition: transform 0.5s;
	}

	#controls-inner,
	#controls-toggle {
		background: #eee;
		border-radius: 10px 10px 0 0;
	}

	.repo-link {
		width: 30px;
		height: 30px;
	}

	#background {
		height: 100vh;
	}

	#DVD {
		width: 256px;
		height: 130px;
		position: relative;
	}

	@media (min-width: 640px) {
		#DVD {
			width: 512px;
			height: 260px;
		}
	}
</style>