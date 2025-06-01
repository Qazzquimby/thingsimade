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
    const rotationAngle = 65; // Degrees for cards not in focus
    const outTranslateX = '100%'; 
    const inTranslateX = '-100%';
    
    let style = {
      transform: `rotateY(0deg) translateX(200%)`, // Default to far right, invisible
      opacity: 0,
      zIndex: 0,
      pointerEvents: 'none'
    };

    if (totalItems === 0) return `opacity: 0; pointer-events: none;`;

    if (cardIndex === activeIndex) {
      // Current card
      style.transform = 'rotateY(0deg) translateX(0%)';
      style.opacity = 1;
      style.zIndex = 2;
      style.pointerEvents = 'auto';
    } else if (cardIndex === (activeIndex + 1 + totalItems) % totalItems) {
      // Next card (visually comes from left)
      style.transform = `rotateY(-${rotationAngle}deg) translateX(${inTranslateX})`;
      style.opacity = 0; // Hidden, ready to animate in
      style.zIndex = 1; // Behind current
    } else if (cardIndex === (activeIndex - 1 + totalItems) % totalItems) {
      // Previous card (visually goes to right)
      style.transform = `rotateY(${rotationAngle}deg) translateX(${outTranslateX})`;
      style.opacity = 0; // Hidden, animated out
      style.zIndex = 1; // Behind current
    }
    // Other cards remain far out and transparent

    return `transform: ${style.transform}; opacity: ${style.opacity}; z-index: ${style.zIndex}; pointer-events: ${style.pointerEvents};`;
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
    transform-origin: 0% 50%; /* Rotate around left edge, vertically centered */
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
