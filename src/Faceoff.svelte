<script>
    import { onMount } from 'svelte';
  
    let windowWidth;
  
    onMount(() => {
      const updateWindowWidth = () => {
        windowWidth = window.innerWidth;
      };
  
      updateWindowWidth();
      window.addEventListener('resize', updateWindowWidth);
  
      return () => {
        window.removeEventListener('resize', updateWindowWidth);
      };
    });
  </script>
  
  {#if windowWidth >= 1200}
    <div class="faceoff-container">
      <div class="faceoff-image left">
        <img src="/kamala.png" alt="Kamala Harris" />
      </div>
      <div class="faceoff-image right">
        <img src="/trump.png" alt="Donald Trump" />
      </div>
    </div>
  {/if}
  
  <style>
    .faceoff-container {
      position: fixed;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      pointer-events: none;
      z-index: 1;
    }
  
    .faceoff-image {
      position: absolute;
      top: 0;
      height: 100vh;
      width: auto;
      overflow: hidden;
    }
  
    .faceoff-image img {
      height: 100%;
      width: auto;
      object-fit: cover;
      object-position: center;
    }
  
    .left {
      left: 0;
      transform: translateX(-20%);
    }
  
    .right {
      right: 0;
      transform: translateX(20%);
    }
  </style>