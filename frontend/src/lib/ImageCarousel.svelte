<script lang="ts">
	import { onMount, onDestroy } from 'svelte';

	// TODO: Fetch images and image preview separately
	const images = [
		'https://gwenliveswell.com/wp-content/uploads/2020/08/Untitled-design-29.png',
		'https://snaped.fns.usda.gov/sites/default/files/seasonal-produce/2018-05/watermelon.jpg',
		'https://gwenliveswell.com/wp-content/uploads/2020/08/Untitled-design-29.png'
	];

    // TODO: make the image padded thing more robust -> less dependent on weird logic, just hard code the empty divs?
	let paddedImages = ['', ...images, ''];
	let previewImgWidth = 300;
	let previewImgMargin = 16;
	const styleDeclaration = `--previewImgWidth: ${previewImgWidth}px; --previewImgMargin: 0px ${previewImgMargin}px`;
	let autoScrollInterval: number;

	// Lightbox state
	let lightboxVisible = false;
	let lightboxImageUrl = '';

    // TODO: investigate if these need to be defined onMount since it uses a ref
	let imageCarouselScrollContainer: HTMLDivElement;
	function snapToNextPicture(): void {
        clearInterval(autoScrollInterval);
		imageCarouselScrollContainer.scrollLeft += previewImgWidth + 2 * previewImgMargin;
	}
	function snapToPrevPicture(): void {
        clearInterval(autoScrollInterval);
		imageCarouselScrollContainer.scrollLeft -= previewImgWidth + 2 * previewImgMargin;
	}
	// Open lightbox with the clicked image
	function openLightbox(imageUrl: string): null {
		lightboxImageUrl = imageUrl;
		lightboxVisible = true;
        return null;
	}

	// Close lightbox
	function closeLightbox(): void {
		lightboxVisible = false;
		lightboxImageUrl = '';
	}
	onMount(() => {
		autoScrollInterval = setInterval(() => {
			snapToNextPicture();
		}, 3000);
	});

	onDestroy(() => {
		clearInterval(autoScrollInterval);
	});
</script>

<!-- TODO: REPLACE TEXT WITH ICONS IN ALL BUTTONS -->
<div style={styleDeclaration} class="carousel-container">
	<button on:click={snapToPrevPicture} class="carousel-button prev" aria-label="Previous Image">
		&lt;
	</button>
	<div class="image-container" bind:this={imageCarouselScrollContainer}>
		<!-- enforce padding between images, and center the main image -->
		{#each paddedImages as image, index}
			{#if image === ''}
				<div class="dummy-img"></div>
			{:else}
            <div class="carousel-image-container">
                <button on:click={() => openLightbox(image)} class="carousel-image-btn-wrapper">
                    <img src={image} class="carousel-image" alt={'carousel image'}  loading="lazy" />
                </button>
            </div>
			{/if}
		{/each}
	</div>
	<button on:click={snapToNextPicture} class="carousel-button next" aria-label="Next Image">
		&gt;
	</button>
</div>

{#if lightboxVisible}
	<div class="lightbox" on:click|self={closeLightbox}>
		<div class="lightbox-content">
			<img src={lightboxImageUrl} alt="lightbox image" class="lightbox-image" />
			<button on:click={closeLightbox} class="lightbox-close" aria-label="Close Lightbox">
				&times;
			</button>
		</div>
	</div>
{/if}

<style>
	.carousel-container {
		display: flex;
		justify-content: center;
		align-items: center;
		width: 100%;
		overflow: hidden;
		position: relative;
	}
	.carousel-image-container,
	.dummy-img {
		scroll-snap-align: center;
		width: 300px;
		height: 200px;
		flex: 0 0 auto;
		margin: var(--previewImgMargin);
	}
    .carousel-image-container {
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        cursor: pointer;
    }
	.carousel-image {
        height:100%;
        width: 100%;
		object-fit: cover;
		transition:
			transform 0.3s ease,
			opacity 0.3s ease;
		cursor: pointer;
        border-radius: 12px;
	}
    .carousel-image-btn-wrapper {
        height: 100%;
    }

	.image-container {
		display: flex;
		flex-direction: row;
		flex-wrap: nowrap;
		-webkit-overflow-scrolling: touch;
		-ms-overflow-style: '-ms-autohiding-scrollbar';
		scroll-behavior: smooth;
		scroll-snap-type: x mandatory;
		scrollbar-width: 0;
		width: 1000px;
		overflow-x: scroll;
		-ms-overflow-style: none; /* IE and Edge */
		scrollbar-width: none; /* Firefox */
		margin: 0px 8px;
		::-webkit-scrollbar {
			display: none;
		}
	}

	.carousel-button {
		background-color: rgba(200, 200, 255, 0.6);
		border: none;
		border-radius: 50%;
		width: 40px;
		height: 40px;
		font-size: 20px;
		cursor: pointer;
		transition: background-color 0.3s ease;
	}

	.carousel-button:hover {
		background-color: rgba(200, 200, 255, 0.9);
	}

	.lightbox {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background-color: rgba(0, 0, 0, 0.8);
		display: flex;
		justify-content: center;
		align-items: center;
		z-index: 1000;
	}

	.lightbox-content {
		position: relative;
		max-width: 90%;
		max-height: 90%;
	}

	.lightbox-image {
		max-width: 100%;
		max-height: 80vh;
		border-radius: 12px;
		box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
	}

	.lightbox-close {
		position: absolute;
		top: -30px;
		right: -30px;
		background-color: rgba(255, 255, 255, 0.8);
		border: none;
		border-radius: 50%;
		width: 40px;
		height: 40px;
		font-size: 24px;
		cursor: pointer;
		display: flex;
		justify-content: center;
	}

	.lightbox-close:hover {
		background-color: rgba(255, 255, 255, 1);
	}
</style>
