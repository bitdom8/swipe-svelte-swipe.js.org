

<script lang="ts">
	import { attach0, attach0add } from '$src/stores/api';

	import { onMount } from 'svelte';

	let slider;
	let slides;


	onMount(async () => {

		(slider = document.querySelector('.slider-container')),
			(slides = Array.from(document.querySelectorAll('.slide')));

		// set up our state
		let isDragging = false,
			startPos = 0,
			currentTranslate = 0,
			prevTranslate = 0,
			animationID,
			currentIndex = 0;

		// add our event listeners
		slides.forEach((slide, index) => {
			const slideImage = slide.querySelector('img');
			// disable default image drag
			slideImage.addEventListener('dragstart', (e) => e.preventDefault());
			// touch events
			slide.addEventListener('touchstart', touchStart(index));
			slide.addEventListener('touchend', touchEnd);
			slide.addEventListener('touchmove', touchMove);
			// mouse events
			slide.addEventListener('mousedown', touchStart(index));
			slide.addEventListener('mouseup', touchEnd);
			slide.addEventListener('mousemove', touchMove);
			slide.addEventListener('mouseleave', touchEnd);
		});

		// make responsive to viewport changes
		window.addEventListener('resize', setPositionByIndex);

		// prevent menu popup on long press
		window.oncontextmenu = function (event) {
			event.preventDefault();
			event.stopPropagation();
			return false;
		};

		function getPositionX(event) {
			return event.type.includes('mouse') ? event.pageX : event.touches[0].clientX;
		}

		// use a HOF so we have index in a closure
		function touchStart(index) {
			return function (event) {
				currentIndex = index;
				startPos = getPositionX(event);
				isDragging = true;
				animationID = requestAnimationFrame(animation);
				slider.classList.add('grabbing');
			};
		}

		function touchMove(event) {
			if (isDragging) {
				const currentPosition = getPositionX(event);
				currentTranslate = prevTranslate + currentPosition - startPos;
			}
		}

		function touchEnd() {
			cancelAnimationFrame(animationID);
			isDragging = false;
			const movedBy = currentTranslate - prevTranslate;

			// if moved enough negative then snap to next slide if there is one
			if (movedBy < -100 && currentIndex < slides.length - 1) currentIndex += 1;

			// if moved enough positive then snap to previous slide if there is one
			if (movedBy > 100 && currentIndex > 0) currentIndex -= 1;

			setPositionByIndex();

			slider.classList.remove('grabbing');
		}

		function animation() {
			setSliderPosition();
			if (isDragging) requestAnimationFrame(animation);
		}

		function setPositionByIndex() {
			currentTranslate = currentIndex * -window.innerWidth;
			prevTranslate = currentTranslate;
			setSliderPosition();
		}

		function setSliderPosition() {
			slider.style.transform = `translateX(${currentTranslate}px)`;
		}
	});
</script>
<!-- {$attach0} -->
<body>
	<div class="slider-container" style="height: {$attach0.length > 3 ? "70vh" : "20vh"}">
		{#each $attach0 as img}
			<div class="slide">
				<img alt={img.split('/')[img.split('/').length - 1]} src={img} />
			</div>
		{/each}
	</div>
</body>

<!-- {$attach0} -->

<style lang="scss" scoped>
	:root {
		--shadow: rgba(0, 0, 0, 0.8);
	}

	* {
		box-sizing: border-box;
		padding: 0;
		margin: 0;
	}

	body {
		height: 100%;
		width: 100%;
		overflow: hidden;
	}

	.slider-container {
		// height: 100vh;
		height: 20vh;
		display: inline-flex;
		overflow: hidden;
		scrollbar-width: none;
		transform: translateX(0);
		will-change: transform;
		transition: transform 0.3s ease-out;
		cursor: grab;
	}

	.slide {
		max-height: 60vh;
		// max-height: 100vh;
		width: 100vw;
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 1rem;
	}

	@media (min-width: 1200px) {
		.slide {
			padding: 3rem;
		}
	}

	.slide img {
		max-width: 100%;
		max-height: 100%;
		transition: transform 0.3s ease-in-out;
		box-shadow: 5px 5px 50px -1px var(--shadow);
		border-radius: 4px;
		user-select: none;
	}

	// .grabbing {
	// 	cursor: grabbing;
	// }

	.grabbing .slide img {
		transform: scale(0.9);
		box-shadow: 5px 5px 40px -1px var(--shadow);
	}
</style>
