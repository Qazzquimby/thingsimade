<script>
  import { tweened } from 'svelte/motion';
  import { quintOut } from 'svelte/easing';

  let items = [
    { id: 0, title: 'Serene Mountain Valley', imageUrl: 'https://picsum.photos/seed/mval1/600/800', description: 'A breathtaking view of a lush green valley nestled between towering snow-capped mountains, under a clear blue sky. Perfect for hiking and nature photography.' },
    { id: 1, title: 'Vibrant City Skylines', imageUrl: 'https://picsum.photos/seed/city2/600/800', description: 'The dazzling lights of a modern metropolis at dusk. Skyscrapers reach for the stars, reflecting in the calm waters below. A testament to human ingenuity.' },
    { id: 2, title: 'Mysterious Deep Ocean', imageUrl: 'https://picsum.photos/seed/ocean3/600/800', description: 'Exploring the enigmatic depths of the ocean, home to bioluminescent creatures and ancient shipwrecks. A world of wonder and discovery awaits.' },
    { id: 3, title: 'Enchanted Forest Path', imageUrl: 'https://picsum.photos/seed/forest4/600/800', description: 'A sun-dappled path winding through an ancient forest. Tall trees form a natural canopy, and the air is filled with the sounds of wildlife.' },
    { id: 4, title: 'Expansive Desert Dunes', imageUrl: 'https://picsum.photos/seed/desert5/600/800', description: 'The stark and beautiful landscape of a vast desert. Golden sand dunes stretch as far as the eye can see, sculpted by the wind into mesmerizing patterns.' }
  ];
  let currentIndex = 0; // Target integer index
  let lastScrollTime = 0;
  const scrollDebounce = 100; // Debounce for rapidly changing the animation target
  const animationDuration = 700; // Duration for the tweened animation

  const displayIndex = tweened(currentIndex, {
    duration: animationDuration,
    easing: quintOut
  });

  // Reactive statement to update tween target when currentIndex changes
  $: if (items.length > 0 && $displayIndex !== currentIndex) {
    displayIndex.set(currentIndex, { interrupt: true });
  }

  function handleWheel(event) {
    const now = Date.now();
    if (now - lastScrollTime < scrollDebounce) {
      return; // Debounce rapid wheel events
    }
    lastScrollTime = now;

    event.preventDefault();
    if (items.length === 0) return;

    if (event.deltaY > 0) {
      // Scroll down
      currentIndex = (currentIndex + 1) % items.length;
    } else {
      // Scroll up
      currentIndex = (currentIndex - 1 + items.length) % items.length;
    }
    // The reactive statement above will trigger displayIndex.set()
  }

  function getCardStyle(cardIndex, activeDisplayIndex, totalItems) {
    if (totalItems === 0) return `opacity: 0; pointer-events: none;`;

    const cardHeightVh = 25;
    const cardMarginVh = 5;
    const slotOffsetVh = cardHeightVh + cardMarginVh;

    const visibleWindow = 2; // Number of cards to show/style above and below the active one
    const maxRotationZ = 20; // Max Z-axis rotation

    let diff = cardIndex - activeDisplayIndex; // activeDisplayIndex is now a float
    // Normalize diff for circular array behavior
    if (Math.abs(diff) > totalItems / 2) {
      diff = diff > 0 ? diff - totalItems : diff + totalItems;
    }

    const currentTranslateY = diff * slotOffsetVh;
    const absDiff = Math.abs(diff);

    let currentRotateZ, currentScale, currentOpacity, currentZIndex, currentPointerEvents;

    // Base calculations for cards within the "active zone"
    if (absDiff <= visibleWindow + 0.5) { // +0.5 to allow smooth transition out of view
        currentRotateZ = (-diff / (visibleWindow + 0.1)) * maxRotationZ;
        currentScale = 1 - (absDiff * 0.05);
        currentOpacity = 1 - (absDiff / (visibleWindow + 0.5));

        // Clamp scale and opacity to avoid negative or excessively small values
        currentScale = Math.max(0.80, currentScale);
        currentOpacity = Math.max(0, currentOpacity);

        // Determine z-index and pointer events based on proximity to the "center"
        if (absDiff < 0.5) { // Card is "most active" / closest to center
            currentZIndex = visibleWindow + 2; // Highest z-index
            currentPointerEvents = 'auto';    // Interactive

            // If very close to center, snap to perfect values for visual crispness
            if (absDiff < 0.01) {
                currentRotateZ = 0;
                currentScale = 1;
                currentOpacity = 1;
            }
        } else { // Cards not in the absolute center slot
            currentZIndex = Math.max(0, visibleWindow - Math.floor(absDiff) + 1); // Ensure zIndex isn't negative
            currentPointerEvents = 'none';
        }

        // Original logic: Further fade cards at extreme angles, but only if not the "center" one
        if (absDiff >= 0.5 && Math.abs(currentRotateZ) > maxRotationZ * 0.9) {
            currentOpacity *= 0.7;
        }
    } else { // Cards far outside the visible window
        currentOpacity = 0;
        currentZIndex = 0;
        currentPointerEvents = 'none';
        // Rotate them more sharply out of view and scale down
        currentRotateZ = diff > 0 ? -maxRotationZ * 1.5 : maxRotationZ * 1.5;
        currentScale = 0.80;
    }
    
    return `transform: translateY(${currentTranslateY}vh) rotate(${currentRotateZ}deg) scale(${currentScale}); opacity: ${currentOpacity}; z-index: ${currentZIndex}; pointer-events: ${currentPointerEvents};`;
  }
</script>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<!-- svelte-ignore a11y-mouse-events-have-key-events -->
<div class="app-container" on:wheel={handleWheel}>
  {#if items.length > 0}
    {#each items as item, i (item.id)}
      <div class="card" style={getCardStyle(i, $displayIndex, items.length)}>
        <div class="card-image">
          <img src={item.imageUrl} alt={item.title} />
        </div>
        <div class="card-content">
          <h2>{item.title}</h2>
          <p>{item.description}</p>
        </div>
      </div>
    {/each}
  {:else}
    <p class="no-items-message">No items to display.</p>
  {/if}
</div>

<style>
  .app-container {
    width: 100vw;
    height: 100vh;
    position: relative;
    perspective: 1800px; /* Increased perspective for a more pronounced 3D effect */
    perspective-origin: 0% 50%; /* Vanishing point to the left */
    overflow: hidden;
    background-color: #2c3e50; /* Match body background or choose another */
  }

  .card {
    position: absolute;
    /* Centering the slot for the active card (diff=0, translateY=0) */
    top: calc(50vh - 12.5vh); /* Assumes card height is 25vh. (50vh - half_card_height) */
    left: 5vw;
    width: 90vw;
    height: 25vh; /* New smaller height */
    display: flex;
    background-color: #ffffff;
    box-shadow: 0 10px 25px rgba(0,0,0,0.25); /* Adjusted shadow */
    border-radius: 8px; /* Slightly smaller radius */
    transform-origin: -150vw 50%; /* Pivot far to the left, adjust as needed. */
    /* transition: transform 0.7s cubic-bezier(0.3, 1, 0.3, 1), opacity 0.6s ease-out; */ /* Removed: tweened store now handles smooth animation */
    will-change: transform, opacity; /* Hint for browser optimization */
    overflow: hidden;
  }

  .card-image {
    flex: 0 0 30%; /* Image takes 30% width */
    max-width: 30%;
    height: 100%;
    overflow: hidden;
  }

  .card-image img {
    width: 100%;
    height: 100%;
    object-fit: cover; /* Cover the area, might crop */
    display: block;
  }

  .card-content {
    flex: 1;
    padding: 20px 25px; /* Adjusted padding */
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    justify-content: center;
    color: #333;
  }

  .card-content h2 {
    margin-top: 0;
    font-size: 1.6em; /* Adjusted for smaller card */
    color: #2c3e50;
    margin-bottom: 10px;
  }

  .card-content p {
    font-size: 0.9em; /* Adjusted for smaller card */
    line-height: 1.5; /* Adjusted line height */
    color: #555;
  }

  .no-items-message {
    color: white;
    font-size: 1.5em;
    text-align: center;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

  /* Responsive adjustments */
  @media (max-width: 768px) {
    .card {
      flex-direction: column; /* Stack image and text on mobile */
      height: 30vh; /* Or 'auto' if content varies a lot, with min/max-height */
      top: calc(50vh - 15vh); /* 50vh - (30vh/2) */
      left: 2.5vw;
      width: 95vw;
      transform-origin: -100vw 50%; /* Less extreme origin for mobile if needed */
    }

    .card-image {
      flex: 0 0 10vh; /* Fixed height for image part on mobile when stacked */
      max-width: 100%;
      width: 100%;
      height: 10vh; 
    }

    .card-content {
      padding: 15px; /* Adjusted padding for mobile */
      flex: 1; 
    }

    .card-content h2 {
      font-size: 1.3em; /* Adjusted for mobile */
      margin-bottom: 8px;
    }

    .card-content p {
      font-size: 0.85em; /* Adjusted for mobile */
      line-height: 1.4;
    }
  }
</style>
