<script>
  let moneyCount = 0;
  let isRaining = false;
  let money = [];
  let manPosition = { x: 0, y: 0 }; // Position of the man image

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
        // Create a new coin/bill at the cursor position
        money = [...money, {
          x: event.clientX + 50, // Adjust for coin/bill size
          y: event.clientY + 50,
          id: Math.random() // Simple unique ID
        }];

        moneyCount+=10;
      }

  }

  function reset() {
    money = [];
    moneyCount = 0;
  }

  // Initialize man position after component mounts
  import { onMount } from 'svelte';
  onMount(() => {
      // Adjust these values as needed based on your layout.  This assumes
      // the "man" will be positioned at the bottom center of the screen.
      manPosition = {
          x: window.innerWidth / 2 - 50,  // Assuming man is 100px wide
          y: window.innerHeight - 150   // Adjust as needed
      };
  });
</script>

<svelte:window on:mousedown="{handleMouseDown}" on:mouseup="{handleMouseUp}" on:mousemove="{handleMouseMove}"></svelte:window>

<main>
<button on:click="{reset}">Reset</button>
  <div class="money-container">  
    <img src="../src/man.png" alt="Little Man" style="position: absolute; left: {manPosition.x}px; bottom: 0;">
    {#each money as coin (coin.id)}
      <span style="position: absolute; left: {coin.x}px; top: {coin.y}px;">$</span>
    {/each}
  </div>

  
</main>

<style>
  main {
    width: 100vw; /* Full viewport width */
    height: 100vh; /* Full viewport height */
    overflow: hidden; /* Hide scrollbars */
    position: relative;
  }
  .money-container {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none; /* Allow clicks to pass through */
  }

    /* Example styling for the money */
  span {
      color: gold;
      font-size: 20px;
      animation: fall 0.5s linear;
  }

    @keyframes fall {
        0% { transform: translateY(-20px); opacity: 0; }
        100% { transform: translateY(100vh); opacity: 1; }
    }

  img {
    width: 100px; /* Adjust as needed */
    height: auto;
  }
</style>