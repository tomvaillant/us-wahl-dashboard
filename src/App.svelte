<script>
	import Logo from './Logo.svelte';
	import National from './National.svelte';
	import GridBackground from './GridBackground.svelte';
  </script>

  <GridBackground />
  <main>
	<Logo />
	<div class="content-wrapper">
		<div class="hero-title">
			<h1>Wer liegt in den Umfragen vorne?</h1>
			<span>Letzte Aktualisierung:</span>
		</div>
	  <National />
	</div>
  </main>
  
  <style>
	main {
	  max-width: 100%;
	  margin: 0 auto;
	}
  
	.content-wrapper {
	  width: 100%;
	  max-width: 60%;
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
  
	@media (max-width: 768px) {
	  .content-wrapper {
		max-width: 100%;
	  }
	  h1 {
		font-size: 20px;
		margin-bottom: 5px;
	  }
	  .hero-title span {
		font-size: 12px;
	  }
	}
  </style>