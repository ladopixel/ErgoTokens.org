<script>
	import Search from './tokens.svelte'
	import Footer from './footer.svelte'
	import Nfts from './nfts.svelte'
	export let menu = 1;
</script>

<main>

	<ul id="menu">
		<li><a href="/" on:click|preventDefault={() => (menu = 1)}>Search</a></li> |
		<li><a href="/" on:click|preventDefault={() => (menu = 2)}>Nfts</a></li>
	</ul>

	{#if menu === 1}
		<Search />
		<Footer />
		{:else if menu === 2}
		<Nfts />
		<Footer />
	{:else}
		<h1>
			Page Not Found
		</h1>
	{/if}
	
</main>


<!--<Nfts />
<Search />
<Footer />-->
