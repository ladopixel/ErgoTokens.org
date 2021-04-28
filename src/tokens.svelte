<script>
	let value = ''
	let response = []
	let informacionCompletaTokens = []
	let selected = ''

	let claseLista = 'list-group-item'
	let claseListaGrupo = 'list-group'

	let claseSelect = 'mb-2 mt-2 form-select form-select-sm'

	function handleInput(){
		value = event.target.value
	}

	// Cada vez que se modifique el valor de value
	$: if (value.length > 4){
		response = fetch(`https://api.ergoplatform.com/api/v1/tokens/search?query=${value}`)
			.then (res => res.json())
			.then (apiResponse => {
				return response = apiResponse.items
		})
		//claseSelect = claseSelect + ' bg-success text-light'
	}

	// Cada vez que se modifique el valor selected
	$: {
		if (selected != '') {
		informacionCompletaTokens = fetch(`https://api.ergoplatform.com/api/v0/assets/${selected}/issuingBox`)
			.then (res => res.json())
			.then (apiResponseToken => {
				return apiResponseToken || []
			})
		}
	}

	function toUtf8String(hex) {
    var str = '';
    for (var i = 0; i < hex.length; i += 2) {
        str += String.fromCharCode(parseInt(hex.substr(i, 2), 16));
    }
    return str;
}
</script>

<svelte:head>
	<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-eOJMYsd53ii+scO/bJGFsiCZc+5NDVN2yr8+0RDqr0Ql0h+rP48ckxlpbzKgwra6" crossorigin="anonymous">
</svelte:head>

<main>
	<nav class="navbar navbar-light bg-dark rounded">
	  <div class="container-fluid">
	    <a class="mb-1"><img src="ergo.png" width="100"></a>
	    <button class="btn-sm btn bg-dark text-secondary border border-secondary mb-1">NFTs</button>
		<input class="form-control mt-1" on:input={handleInput} placeholder="Your token name (5 letters min)" value={value}>
	  </div>
	</nav>

	<select bind:value={selected} class={claseSelect}>
	{#await response}
		<p>searching...</p>
	{:then response}
		{#if response.length > 0}

			{#each response as token}
				<option value={token.id}>{token.name}</option>
			{/each}
		{:else if response.length > 2}
			<span>No tokens</span>
		{/if}
	{:catch error}
		<p>✖️Something went wrong: {error.message}</p>
	{/await}
	</select>

	{#await informacionCompletaTokens}
		<span>Loading...</span>
	{:then informacionCompletaTokens}
		{#if informacionCompletaTokens.length > 0}
			<ul class={claseListaGrupo}>
				<li class="list-group-item bg-dark text-light">Tokens info</li>
			{#each informacionCompletaTokens as ImagenToken}
				{#if ImagenToken.additionalRegisters.R9}
					<li class={claseLista}>
						<span><a href={toUtf8String(ImagenToken.additionalRegisters.R9).substr(2)} target="_blank"><img src={toUtf8String(ImagenToken.additionalRegisters.R9).substr(2)} class="img-thumbnail" width="200" /></a></span>
					</li>
				{:else}
					<li class={claseLista}>
						<span>Sin imagen</span>
					</li>
				{/if}
				
				<li class={claseLista}>
					<span><small><strong>Id: </strong>{ImagenToken.id}</span>
				</li>
				<li class={claseLista}>
					<span><small><strong>Tx id: </strong>{ImagenToken.txId}</span>
				</li>
				<li class={claseLista}>
					<span><strong>Value: </strong>{ImagenToken.value}</span>
				</li>
				<li class={claseLista}>
					<span><small><strong>Creation Height: </strong>{ImagenToken.creationHeight}</span>
				</li>
				<li class={claseLista}>
					<span><small><strong>Ergo Tree: </strong>{ImagenToken.ergoTree}</span>
				</li>
				<li class={claseLista}>
					<span><small><strong>Address: </strong>{ImagenToken.address}</span>
				</li>
				<li class={claseLista}>
					<span><small><strong>Token Id: </strong>{ImagenToken.assets[0].tokenId}</span>
				</li>
				<li class={claseLista}>
					<span><small><strong>Amount: </strong>{ImagenToken.assets[0].amount}</span>
				</li>
				<li class={claseLista}>
					<span><small><strong>Name: </strong>{ImagenToken.assets[0].name}</span>
				</li>
				<li class={claseLista}>
					<span><small><strong>Decimals: </strong>{ImagenToken.assets[0].decimals}</span>
				</li>
				<li class={claseLista}>
					<span><small><strong>Type: </strong>{ImagenToken.assets[0].type}</span>
				</li>
				<li class="list-group-item bg-dark text-light">Token aditionals registers</li>
				<li class={claseLista}>
					<span><small><strong>R4: </strong>{ImagenToken.additionalRegisters.R4}</span>
				</li>
				<li class={claseLista}>
					<span><small><strong>R5: </strong>{ImagenToken.additionalRegisters.R5}</span>
				</li>
				<li class={claseLista}>	
					<span><small><strong>R6: </strong>{ImagenToken.additionalRegisters.R6}</span>
				</li>
				<li class={claseLista}>	
					<span><small><strong>R7: </strong>{ImagenToken.additionalRegisters.R7}</span>
				</li>
				<li class={claseLista}>
					<span><small><strong>R8: </strong>{ImagenToken.additionalRegisters.R8}</span>
				</li>
				<li class={claseLista}>
					<span><small><strong>R9Cod: </strong>{ImagenToken.additionalRegisters.R9}</small></span>
				</li>
				<li class={claseLista}>
					<span><small><strong>R9Dec: </strong> <a href={toUtf8String(ImagenToken.additionalRegisters.R9).substr(2)}>{toUtf8String(ImagenToken.additionalRegisters.R9).substr(2)}</a></small></span>
				</li>
			{/each}
			</ul>
		{/if}
	{:catch error}
		<p>✖️ Something went wrong: {error.message}</p>
	{/await}
</main>