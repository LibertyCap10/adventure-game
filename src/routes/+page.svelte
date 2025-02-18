<script>
  // import { $state, $derived, $effect } from 'svelte';
  import { onMount } from "svelte";

  let wood = $state(0);
  let woodPerClick = $state(1);
  let workers = $state(0);
  let particles = $state([]);
  let progress = $state(0);

  // Derived state: workers produce 0.5 wood per second each
  const woodPerSecond = $derived(workers * 0.5);

  function chop() {
    wood += 1;
    particles.push({ id: Date.now(), x: Math.random() * 50 - 25 });

    // Remove particle after animation
    setTimeout(() => {
      particles.shift();
    }, 1000);
  }

  function buyWorker() {
    if (wood >= 10) {
      wood -= 10;
      workers += 1;
    }
  }

  function buyIronAxe() {
    if (wood >= 50) {
      wood -= 50;
      woodPerClick *= 1.25;
    }
  }

  // Correct effect: Ensure state updates inside setInterval
  $effect(() => {
    if (workers > 0) {
      // Avoid unnecessary intervals
      const interval = setInterval(() => {
        wood += woodPerSecond;
        progress = (progress + 10) % 100; // Loop animation

      }, 1000);

      return () => clearInterval(interval); // Cleanup on destroy
    }
  });

  onMount(() => {
    const savedWood = localStorage.getItem("wood");
    if (savedWood) wood = parseInt(savedWood);

    const savedWorkers = localStorage.getItem("workers");
    if (savedWorkers) workers = parseInt(savedWorkers);
  });

  $effect(() => {
    localStorage.setItem("wood", wood);
    localStorage.setItem("workers", workers);
  });
</script>





<div class="flex flex-col items-center mt-20">
  <img
    src="/tree.webp"
    alt="Tree"
    class="w-32 h-32 cursor-pointer transition-transform active:scale-90"
    onclick={chop}
  />
  <p>You have {wood} logs</p>
</div>

<br /><br />

<div class="relative flex flex-col items-center">
  <div class="w-64 bg-gray-300 h-4 rounded overflow-hidden">
    <div
      class="bg-blue-500 h-full transition-all"
      style="width: {progress}%"
    ></div>
  </div>

  <p>Workers: {workers} | Wood per sec: {woodPerSecond}</p>
  <button onclick={buyWorker} class="bg-blue-500 p-2 text-white">
    Hire Worker (10 wood)
  </button>

  <br /><br />

  <button
    onclick={buyIronAxe}
    class="bg-yellow-500 p-4 text-white"
    disabled={wood < 50}
  >
    Buy Iron Axe (50 wood)
  </button>
</div>






<style>
  @keyframes float {
    0% {
      transform: translateY(0);
      opacity: 1;
    }
    100% {
      transform: translateY(-30px);
      opacity: 0;
    }
  }
  .animate-float {
    animation: float 1s ease-out;
  }
</style>
