<script>
	import Logo from './Logo.svelte';
	import National from './National.svelte';
  </script>

  <main>
	<Logo />
	<div class="content-wrapper">
	  <h1>Who's leading the polls?</h1>
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
  
	h1 {
	  color: #333;
	  font-size: 24px;
	  margin-bottom: 20px;
	  text-align: center;
	}
  
	@media (max-width: 768px) {
	  .content-wrapper {
		max-width: 100%;
	  }
	}
  </style>