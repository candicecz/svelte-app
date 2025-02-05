<script>
  import Lazy from 'svelte-lazy';
  import { debounce } from '$lib/utils/helpers';

  export let slidesData;
  export let title;
  export let title2;
  export let titleFontClassName;
  export let title2FontClassName = '';

  // [isMobile] means the user is on a mobile device AND in landscape mode.
  export let isMobile = false;

  // Inner width and height of window.
  export let innerWidth;
  export let innerHeight;

  // [sliderEl] is the element that wraps all the project slides and translates based on which slide is active (e.g. slides[active_index]).
  let sliderEl;

  /*
  In mobile, the text slides are condensed into one. So the [slides] array is shortened to only have the first text slide.
  In the html, we iterate over the leftover text slides slidesData.slice(text_slides_index, slidesData.length) and input them to the main text slide.
   */
  $: slides = slidesData;
  $: text_slide_index = slidesData.findIndex((s) => s.type === 'text');
  $: slides = isMobile && slidesData ? slidesData.slice(0, text_slide_index + 1) : slidesData;

  $: slide_els = []; // all slide elements
  $: active_index = 0; // index of currently displayed slide.

  const Cursors = {
    RIGHT: 'right-cursor',
    LEFT: 'left-cursor'
  };

  let sliderCursor = 'cursor';
  let wrapperWidth = 0;

  function handleMousemove(event) {
    const cursorXPosition = event.clientX;
    sliderCursor = wrapperWidth / 2 <= cursorXPosition ? Cursors.RIGHT : Cursors.LEFT;
  }

  /*
    - SLIDE CHANGE  ------------------------
    Assuming that the desktop slides = 100vw and mobile slides have a 5:8 aspect ratio
    */
  let aspectRatio = { mobile: 8 / 5 };
  $: mobile_width = Math.round(innerHeight * aspectRatio.mobile); //in px
  $: DESKTOP_WIDTH = 100; //in vw

  // Calculates the offset for the x translation.
  let calculateXOffset = (w) => {
    // [offset] is calculated by multiplying the width by the active index.
    let offset = active_index * w;
    // When on mobile, the last slide has the image from the previous slide showing on the left.
    if (isMobile && active_index === slides.length - 1) {
      // calculate the left padding by taking the width of the window - the current slide width
      let left_padding = innerWidth - mobile_width;
      offset = offset - left_padding;
    }
    return 0 - offset;
  };

  // Handles the translation on [sliderEl] to move to the next slide.
  const nextSlide = () => {
    // On last slide of project, go to next project index.
    if (isMobile && active_index === slides.length - 1) {
      return updateProjectIndex(id + 1);
    }
    // Update active index.
    active_index = active_index < slides.length - 1 ? active_index + 1 : 0;
    let translate_x = isMobile
      ? `${calculateXOffset(mobile_width)}px`
      : `${calculateXOffset(DESKTOP_WIDTH)}vw`;

    return (sliderEl.style.transform = `translateX(${translate_x})`);
  };

  // Handles the translation on [sliderEl] to move to the previous slide.
  const prevSlide = () => {
    // On last slide of project, go to previous project index.
    if (isMobile && active_index === 0) {
      return updateProjectIndex(id - 1);
    }
    // Update active index.
    active_index = active_index === 0 ? slides.length - 1 : active_index - 1;
    let translate_x = isMobile
      ? `${calculateXOffset(mobile_width)}px`
      : `${calculateXOffset(DESKTOP_WIDTH)}vw`;

    return (sliderEl.style.transform = `translateX(${translate_x})`);
  };

  // Handles the users click depending on which side of the page it's on.
  const handleSliderClick = () => {
    if (sliderCursor === Cursors.RIGHT) {
      nextSlide();
    } else if (sliderCursor === Cursors.LEFT) {
      prevSlide();
    }
  };

  // -------------------------

  // - MOBILE SCROLL -----------------------

  export let updateProjectIndex;
  export let id;

  const handleScrollDown = debounce((e) => {
    const { scrollHeight, scrollTop } = e.target;
    // Detect if user is at the bottom of the window to move to next slide.
    if (scrollTop + innerHeight >= scrollHeight - 100) {
      nextSlide();
    }
  }, 100);
  // -------------------------
</script>

<div
  class="slider-wrapper"
  bind:clientWidth={wrapperWidth}
  on:mousemove={handleMousemove}
  style={isMobile ? `width:${innerWidth}px; height:${innerHeight}px` : ''}
>
  <img class="image-logo" src="images/LOGO-Ai small_Super Bonjour smaller.svg" alt="Logo" />
  <img class="image-logo mobile" src="images/LOGOFACE-Ai.svg" alt="Logo" />
  <h2 class="slider-title">
    <span class={`${titleFontClassName || ''}`}>{title}</span>
    <span class={`${title2FontClassName || ''}`}>{title2 || ''}</span>
  </h2>
  <div
    class={`slider ${sliderCursor}`}
    bind:this={sliderEl}
    on:click={handleSliderClick}
    style={`width: ${isMobile ? `${mobile_width * slides.length}px` : `${slides.length * 100}vw`}`}
  >
    {#each slides as slide, i}
      <div
        class="slider-slide"
        type={slide.type}
        bind:this={slide_els[i]}
        style={`width:${isMobile ? `${Math.round(innerHeight * aspectRatio.mobile)}px` : '100vw'};`}
      >
        {#if slide.type === 'image'}
          <Lazy height={300}>
            <div
              id={i}
              class={`slide ${sliderCursor}`}
              style={`background-position: ${i}00vw center; background-image: url(${slide.src})`}
            />
          </Lazy>
        {:else if slide.type === 'video'}
          <div
            id={i}
            class={`slide slide-video ${sliderCursor} ${
              slide.addPadding ? 'slide-video-extra-padding' : ''
            }`}
            style={`background-position: ${i}00vw center; position: relative;${
              isMobile ? '' : 'width:100vw'
            }`}
          >
            <div class={`${slide.addPadding ? 'video-container' : ''} ${isMobile ? '100%' : ''}`}>
              <!-- svelte-ignore a11y-media-has-caption -->
              <Lazy height={300}>
                <video src={slide.src} autoplay="true" loop muted playsinline />
              </Lazy>
            </div>
          </div>
        {:else if slide.type === 'two-columns'}
          <div class="slide two-columns-slide">
            <div class="slide-column slide-left-column">
              <video src={slide.videoSrc} autoplay="true" loop muted playsinline />
            </div>
            <div class="slide-column slide-right-column">
              <!-- svelte-ignore a11y-img-redundant-alt -->
              <Lazy height={300}>
                <img src={slide.imageSrc} alt="left column image" />
              </Lazy>
            </div>
          </div>
        {:else}
          <div
            class="slide text_slide"
            style={`
            background-color: ${slide.backgroundColor}; color:${slide.color};
            font-family: ${slide.font || 'moret'};
            font-size: ${slide.fontSize};
            overflow-y:${isMobile ? 'scroll' : 'auto'};
            width:100%;
            `}
            on:scroll={handleScrollDown}
          >
            {#if isMobile}
              <!-- All the text is encapsulated in one slide for mobile -->
              {#each slidesData.slice(text_slide_index, slidesData.length) as textSlide}
                <div style="padding:1rem 2rem">
                  <h5 class="text_title">
                    {textSlide.title}
                  </h5>
                  <p>{@html textSlide.src}</p>
                </div>
              {/each}
            {:else}
              <div class="text_slide_container">
                <h5 class="text_title">
                  {slide.title}
                </h5>
                {@html slide.src}
              </div>
            {/if}
          </div>
        {/if}
      </div>
    {/each}
  </div>
  <div class="paginator">
    <h4>{active_index + 1} / {slides.length}</h4>
  </div>
</div>

<style>
  video::-webkit-media-controls-fullscreen-button,
  video::-webkit-media-controls-play-button,
  video::-webkit-media-controls-pausebutton {
    display: none;
  }

  .video-container {
    position: absolute;
    padding-bottom: 56.25%;
    padding-top: 0;
    height: 0;
    overflow: hidden;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

  .video-container iframe,
  .video-container object,
  .video-container embed {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
  }

  .image-logo.mobile {
    display: none;
  }

  .slider-title .roc-grotesk {
    font-family: 'roc-grotesk', sans-serif;
  }

  .slide-video-extra-padding video {
    width: 70vw;
    height: 70vh;
    margin-left: 15vw;
    margin-right: 15vw;
    object-fit: contain;
    margin-top: 15vh;
    margin-bottom: 15vh;
  }
  .slide-video-extra-padding {
    background-color: #c374f6;
  }

  .slide-column {
    flex-shrink: 1;
    background-color: #c374f6;
    width: 25vw;
    min-height: 300px;
    display: flex;
    align-items: center;
  }

  .slide-right-column {
    margin-left: 15px;
  }

  .slide-left-column {
    margin-right: 15px;
  }

  .slide-left-column video {
    width: 100%;
    height: auto;
  }

  .slide-right-column img {
    width: 100%;
    height: auto;
  }

  .two-columns-slide {
    display: flex;
    background-color: #c374f6;
    flex-direction: row;
    flex-wrap: nowrap;
    justify-content: center;
    align-items: center;
    align-content: stretch;
  }
  .text_slide_container {
    padding-left: 55px;
    padding-top: 50px;
    width: 80%;
    line-height: 130%;
  }

  .text_slide_container_mobile {
    width: 100%;
    padding: 2rem;
  }

  .text_title {
    padding-top: 50px;
    padding-bottom: 10px;
    margin-bottom: 2px;
    color: #e2ee75;
    font-size: 18px;
    font-weight: 100;
    font-family: 'Opposit-Medium';
    line-height: 39px;
  }

  .text_slide {
    background-color: #290b15;
    height: 100vh;
    width: 100vw;
    color: white;
    text-align: 15vw;
    font-size: 50px;
    font-weight: 100;
    font-family: 'moret';
  }

  .slider {
    position: relative;
    display: flex;
    flex-wrap: nowrap;
    transition: all 200ms linear 0s;
    background: #c3862c;
  }

  .slide {
    height: 100vh;
    background-size: cover;
    transition: all 200ms ease-out 0s;
  }

  .slider-wrapper {
    position: relative;
    display: flex;
    flex-wrap: nowrap;
    width: 100vw;
    overflow: hidden;
    transition: 0.3s all;
    background: #c3862c;
  }

  .arrow {
    font-family: 'Roc Wide';
    position: absolute;
    width: 70px;
    cursor: pointer;
    top: 0;
    bottom: 0;
    margin: auto;
    color: #fff;
  }

  .left {
    left: 0;
  }

  .right {
    right: 0;
  }

  .paginator {
    position: absolute;
    bottom: 0;
    right: 20vw;
    text-align: center;
    width: 100px;
    font-family: 'moret';
    /* background-color: rgb(255 255 255 / 80%); */
    color: #fff;
    font-size: 36px;
  }

  .slider-title {
    font-family: 'moret';
    /* background-color: rgb(255 255 255 / 80%); */
    min-width: 20vw;
    padding: 25px;
    padding-left: 55px;
    padding-bottom: 30px;
    margin-bottom: 20px;
    position: absolute;
    z-index: 1;
    left: 0px;
    bottom: 0;
    color: #fff;
    font-size: 38px;
    font-weight: normal;
  }
  video {
    width: 100vw;
    height: 100vh;
    object-fit: cover;
  }

  .image-logo {
    position: fixed;
    left: 55px;
    top: 30px;
    width: 177px;
    height: 4.75rem;
    z-index: 1;
    cursor: url(/images/home-cursor.png), auto;
  }

  .right-cursor {
    cursor: url(/images/right-cursor.svg), auto;
  }
  .left-cursor {
    cursor: url(/images/left-cursor.svg), auto;
  }

  @media screen and (max-width: 1200px) {
    .slider-title {
      bottom: 0;
      margin-bottom: -10px;
      font-size: 2.25rem;
    }
    .paginator {
      font-size: 1.6rem;
      right: 20vw;
      bottom: 0;
      margin-bottom: -10px;
    }

    .slide {
      background-attachment: scroll;
      background-position: center !important;
    }

    .image-logo {
      width: 177px;
    }
  }

  @media screen and (max-width: 600px) {
    video {
      width: 100vw;
      height: auto;
      object-fit: unset;
    }

    .slide-right-column {
      margin-left: 10px;
    }

    .slide-left-column {
      margin-right: 10px;
    }
    .slide-video-extra-padding video {
      width: 75vw;
      height: auto;
      margin: 20px 50px;
    }

    .image-logo {
      display: none;
    }
    .image-logo.mobile {
      display: block;
    }

    .slider-title {
      font-size: 1rem;
      width: 5vw;
      padding-left: 25px;
    }

    .paginator {
      font-size: 1rem;
      right: 5vw;
      margin-right: 35px;
    }

    .slide {
      height: 60vw;
    }

    .text_slide {
      padding-top: 0px;
      font-size: 10px !important;
    }

    .text_slide_container {
      padding-top: 5px;
      padding-left: 25px;
    }

    .text_title {
      padding-top: 5px;
      font-size: 18px;
      margin-top: 0;
    }

    .image-logo {
      width: 117px;
      top: 10px;
      left: 25px;
    }
  }
  @media screen and (orientation: landscape) and (max-height: 499px) {
    /* For mobile-size but acts on desktop as well */
  }
</style>
