<script>
	import Logo from './Logo.svelte';
	import National from './National.svelte';
	import SwingStates from './SwingStates.svelte';
	import GridBackground from './GridBackground.svelte';
	import Faceoff from './Faceoff.svelte';
  </script>

  <Faceoff />
  <GridBackground />
  <main>
	<Logo />
	<div class="content-wrapper">
		<div class="hero-title">
			<h1>Wer liegt in den Umfragen vorne?</h1>
			<span>Letzte Aktualisierung:</span>
		</div>
	  <National />
	  <SwingStates />
	</div>
  </main>
  
  <style>
	main {
	  max-width: 100%;
	  margin: 0 auto;
	}

	.hero-title {
		text-align: center;
		margin-top: 6vh;
		margin-bottom: 6vh;
	}

	.hero-title span {
		font-size: 14px;
		opacity: 0.7;
	}
  
	h1 {
	  color: #07184D;
	  font-size: 32px;
	  font-weight: bold;
	  margin-bottom: 10px;
	}

	.content-wrapper {
  width: 100%;
  margin: 0 auto;
  padding: 0 20px; /* Add some padding for small screens */
  box-sizing: border-box; /* Ensure padding is included in the width */
}

	/* Mobile devices */
	@media (max-width: 767px) {
	.content-wrapper {
		max-width: 95%;
	}
	}

	/* Tablets and small desktops */
	@media (min-width: 768px) and (max-width: 1199px) {
	.content-wrapper {
		max-width: 80%;
	}
	}

	/* Normal desktops */
	@media (min-width: 1200px) and (max-width: 1799px) {
	.content-wrapper {
		max-width: 50%;
	}
	}

	/* Large monitors */
	@media (min-width: 1800px) {
	.content-wrapper {
		max-width: 50%;
	}
	}
  
	@media (max-width: 768px) {
	  h1 {
		font-size: 20px;
		margin-bottom: 5px;
	  }
	  .hero-title span {
		font-size: 12px;
	  }
	}
  </style>