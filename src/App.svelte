<script>
  let items = [
    { id: 0, title: 'Serene Mountain Valley', imageUrl: 'https://picsum.photos/seed/mval1/600/800', description: 'A breathtaking view of a lush green valley nestled between towering snow-capped mountains, under a clear blue sky. Perfect for hiking and nature photography.' },
    { id: 1, title: 'Vibrant City Skylines', imageUrl: 'https://picsum.photos/seed/city2/600/800', description: 'The dazzling lights of a modern metropolis at dusk. Skyscrapers reach for the stars, reflecting in the calm waters below. A testament to human ingenuity.' },
    { id: 2, title: 'Mysterious Deep Ocean', imageUrl: 'https://picsum.photos/seed/ocean3/600/800', description: 'Exploring the enigmatic depths of the ocean, home to bioluminescent creatures and ancient shipwrecks. A world of wonder and discovery awaits.' },
    { id: 3, title: 'Enchanted Forest Path', imageUrl: 'https://picsum.photos/seed/forest4/600/800', description: 'A sun-dappled path winding through an ancient forest. Tall trees form a natural canopy, and the air is filled with the sounds of wildlife.' },
    { id: 4, title: 'Expansive Desert Dunes', imageUrl: 'https://picsum.photos/seed/desert5/600/800', description: 'The stark and beautiful landscape of a vast desert. Golden sand dunes stretch as far as the eye can see, sculpted by the wind into mesmerizing patterns.' }
  ];
  let currentIndex = 0;
  let lastScrollTime = 0;
  const scrollCooldown = 700; // milliseconds for transition + buffer

  function handleWheel(event) {
    const now = Date.now();
    if (now - lastScrollTime < scrollCooldown) {
      return; // Debounce/throttle scroll
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
  }

  function getCardStyle(cardIndex, activeIndex, totalItems) {
    if (totalItems === 0) return `opacity: 0; pointer-events: none;`;

    const slotHeightVh = 90; // Effective height for positioning each card slot vertically (matches card height)
    const visibleWindow = 1; // Number of cards to show/style above and below the active one (e.g., 1 means active + 1 above + 1 below)
    const maxRotationX = 35; // Max X-axis rotation for cards at the edge of the visible window (degrees)

    let diff = cardIndex - activeIndex;
    // Normalize diff for circular array behavior
    if (Math.abs(diff) > totalItems / 2) {
      diff = diff > 0 ? diff - totalItems : diff + totalItems;
    }

    const currentTranslateY = diff * slotHeightVh;
    let currentRotateX = 0;
    let currentScale = 1;
    let currentOpacity = 0;
    let currentZIndex = 0;
    let currentPointerEvents = 'none';

    if (diff === 0) { // Current card (center of viewport)
        currentRotateX = 0;
        currentScale = 1;
        currentOpacity = 1;
        currentZIndex = visibleWindow + 2; // Highest z-index
        currentPointerEvents = 'auto';
    } else if (Math.abs(diff) <= visibleWindow) { // Visible adjacent cards
        // Rotation is proportional to how far off-center the card is (diff)
        // Positive diff (below center) = positive rotateX (top edge towards viewer)
        // Negative diff (above center) = negative rotateX (bottom edge towards viewer)
        currentRotateX = (diff / (visibleWindow + 0.1)) * maxRotationX; // +0.1 to ensure diff=0 gives 0 rotation
        currentScale = 1 - (Math.abs(diff) * 0.03); // Slightly scale down cards further away
        currentOpacity = 1 - (Math.abs(diff) / (visibleWindow + 0.7)); // Fade out cards further away (+0.7 for smoother fade)
        currentZIndex = visibleWindow - Math.abs(diff) + 1;
        
        // Further reduce opacity for cards rotated significantly, enhancing the "depth"
        if (Math.abs(currentRotateX) > maxRotationX * 0.8) { 
             currentOpacity *= 0.8;
        }
    } else { // Cards outside the immediate visible window
        currentOpacity = 0; // Make them invisible
        currentZIndex = 0;
        // Position them far off and rotate sharply to be out of view
        currentRotateX = diff > 0 ? 60 : -60; // Steeper angle for far cards
        currentScale = 0.85; // Scale them down more
    }
    
    // Final check to hide cards rotated too extremely (e.g., edge-on or facing away)
    if (Math.abs(currentRotateX) >= 80) {
        currentOpacity = Math.min(currentOpacity, 0.05); // Keep very faint or hide
    }
    if (Math.abs(currentRotateX) >= 90) { // Definitely hide if 90 degrees or more
        currentOpacity = 0; 
    }

    return `transform: translateY(${currentTranslateY}vh) rotateX(${currentRotateX}deg) scale(${currentScale}); opacity: ${currentOpacity}; z-index: ${currentZIndex}; pointer-events: ${currentPointerEvents};`;
  }
</script>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<!-- svelte-ignore a11y-mouse-events-have-key-events -->
<div class="app-container" on:wheel={handleWheel}>
  {#if items.length > 0}
    {#each items as item, i (item.id)}
      <div class="card" style={getCardStyle(i, currentIndex, items.length)}>
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
    top: 5vh; /* Add some margin from top/bottom */
    left: 5vw; /* Add some margin from left/right */
    width: 90vw;
    height: 90vh;
    display: flex;
    background-color: #ffffff;
    box-shadow: 0 15px 35px rgba(0,0,0,0.3);
    border-radius: 10px;
    transform-origin: 0% 50%; /* Rotate around the card's own left edge, vertically centered */
    transition: transform 0.7s cubic-bezier(0.3, 1, 0.3, 1), opacity 0.6s ease-out;
    will-change: transform, opacity;
    overflow: hidden; /* Ensure content within card respects border-radius */
  }

  .card-image {
    flex: 0 0 45%; /* Image takes 45% width */
    max-width: 45%;
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
    flex: 1; /* Text takes remaining space */
    padding: 30px 40px;
    overflow-y: auto; /* Scroll if content is too long */
    display: flex;
    flex-direction: column;
    justify-content: center;
    color: #333;
  }

  .card-content h2 {
    margin-top: 0;
    font-size: 2.2em;
    color: #2c3e50;
    margin-bottom: 20px;
  }

  .card-content p {
    font-size: 1.1em;
    line-height: 1.7;
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
      top: 2.5vh;
      left: 2.5vw;
      width: 95vw;
      height: 95vh;
    }

    .card-image {
      flex: 0 0 40vh; /* Image takes fixed height portion */
      max-width: 100%;
      width: 100%;
      height: 40vh; /* Explicit height */
    }

    .card-content {
      padding: 20px;
      flex: 1; /* Takes remaining height */
    }

    .card-content h2 {
      font-size: 1.6em;
      margin-bottom: 10px;
    }

    .card-content p {
      font-size: 0.95em;
      line-height: 1.6;
    }
  }
</style>
