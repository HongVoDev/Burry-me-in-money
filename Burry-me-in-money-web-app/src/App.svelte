<script>
  import { onMount } from 'svelte';

  let isRaining = false;
  let coins = [];
  let manPosition = { x: 0, y: 0 }; // Position of the man image
  const coinSize = 25; // Adjust as needed
  const groundLevel = 0.8; // Percentage of screen height that represents the ground
  const coinImageSrc = '../src/coin.png'; // Path to your coin image
  let accumulatedCoins = 0; // Keep track of coins to batch creation
  const coinBatchSize = 5;    // Create coins in batches
  const initialStackHeight = 100; // Number of coins to initially stack on the ground
  const secondaryStackStart = 55; // Number of coins before secondary stack begins

  function startRain() {
    isRaining = true;
  }

  function stopRain() {
    isRaining = false;
  }

  function handleMouseDown(event) {
    startRain();
  }

  function handleMouseUp(event) {
    stopRain();
  }

  function handleMouseMove(event) {
    if (isRaining) {
      accumulatedCoins++;
      if (accumulatedCoins >= coinBatchSize) {
        accumulatedCoins = 0;
        // Create multiple coins in a small burst
        for (let i = 0; i < coinBatchSize; i++) {
          createCoin(event.clientX, event.clientY);
        }
      }
    }
  }

  function createCoin(x, y) {
    const spread = 20; // Random spread around the cursor
    const randomX = x + (Math.random() - 0.5) * spread;
    const randomY = y + (Math.random() - 0.5) * spread;

    coins = [...coins, {
      x: randomX,
      y: randomY,
      vx: (Math.random() - 0.5) * 5, // Initial horizontal velocity
      vy: Math.random() * -10, // Initial upward velocity for bounce
      gravity: 0.5, // Adjust gravity as needed
      id: Math.random(), // Simple unique ID
      rotation: Math.random() * 360, // Initial random rotation
      rotationSpeed: (Math.random() - 0.5) * 10, // Rotation speed
      isSettled: false,
      stackLevel: 0 // Add a stack level property
    }];
  }


  function reset() {
    coins = [];
  }

  // Initialize man position after component mounts
  onMount(() => {
    // Adjust these values as needed based on your layout.  This assumes
    // the 'man' will be positioned at the bottom center of the screen.
    manPosition = {
      x: window.innerWidth / 2 - 50,  // Assuming man is 100px wide
      y: window.innerHeight * groundLevel - 100,   // Adjust as needed
    };
  });


  // Game Loop (Physics Simulation)
  function updatePhysics() {
    coins = coins.map((coin, index) => {
      if (!coin.isSettled) {
        coin.vy += coin.gravity; // Apply gravity
        coin.x += coin.vx;      // Apply horizontal velocity
        coin.y += coin.vy;      // Apply vertical velocity
        coin.rotation += coin.rotationSpeed; // Rotate coin

        // Ground collision logic
        let adjustedGroundLevel = window.innerHeight - coinSize; // Default ground

        if (coins.length > initialStackHeight) {
            if (index >= initialStackHeight) {
              coin.stackLevel = Math.floor((index - initialStackHeight) / 5);
              adjustedGroundLevel -= coin.stackLevel * 0.5; // Adjust ground for stack
            }
        }
      
        // Ground collision
        if (coin.y > adjustedGroundLevel) {
          coin.y = adjustedGroundLevel;
          coin.vy *= -0.5; // Bounce (reduce velocity)
          coin.vx *= 0.8; // Dampen horizontal velocity

          if (Math.abs(coin.vy) < 1 && Math.abs(coin.vx) < 1) {
            coin.isSettled = true; // Coin has settled
          }
        }

        //Boundary collision (bounce off walls)
        if (coin.x < 0) {
          coin.x = 0;
          coin.vx *= -0.5;
        } else if (coin.x > window.innerWidth - coinSize) {
          coin.x = window.innerWidth - coinSize;
          coin.vx *= -0.5;
        }
      }
      return coin;
    });
  }


  // Run the physics update every frame
  setInterval(updatePhysics, 16); // ~60 FPS

</script>

<svelte:window
  on:mousedown="{handleMouseDown}"
  on:mouseup="{handleMouseUp}"
  on:mousemove="{handleMouseMove}"
/>

<main>
  <button on:click="{reset}">Reset</button>
  <div class="money-container">
    <img
      src='../src/man.png'
      alt='Little Man'
style:position='absolute'
style:left='{manPosition.x}px'
style:bottom='0'
style:zIndex='10'    />
    {#each coins as coin (coin.id)}
      <img
        src='{coinImageSrc}'
        alt='Coin'
        style:position='absolute'
        style:left='{coin.x}px'
        style:top='{coin.y}px'
        style:width='{coinSize}px'
        style:height='{coinSize}px'
        style:transform='rotate({coin.rotation}deg)'
        style:z-index='5'
        style:opacity='{coin.isSettled ? 0.8 : 1}'
       style:transition='opacity 0.3s ease-in-out'
      />
    {/each}
  </div>
</main>

<style>
  main {
    width: 100vw; /* Full viewport width */
    height: 100vh; /* Full viewport height */
    overflow: hidden; /* Hide scrollbars */
    position: relative;
    background-color: #444; /* Dark background */
  }

  .money-container {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none; /* Allow clicks to pass through */
  }

  img {
    width: 100px; /* Adjust as needed */
    height: auto;
  }
</style>
