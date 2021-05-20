<script>
	let value = ''
	let response = []
	let informacionCompletaTokens = []
	let selected = ''

	let claseLista = 'list-group-item'
	let claseListaGrupo = 'list-group'

	let claseSelect = 'mb-2 mt-2 form-select form-select-sm '

	function cambiarDeColorSelect(){
		claseSelect = claseSelect + 'bg-info text-light'
	}

	function handleInput(){
		value = event.target.value
	}

	// Cada vez que se modifique el valor de value
	$: if (value.length > 4){
		claseSelect = 'mb-2 mt-2 form-select form-select-sm '
		response = fetch(`https://api.ergoplatform.com/api/v1/tokens/search?query=${value}`)
			.then (res => res.json())
			.then (apiResponse => {
				return response = apiResponse.items
		})
	}

	// Cada vez que se modifique el valor selected
	$: {
		if ((selected != '') && (selected !=0)) {
		informacionCompletaTokens = fetch(`https://api.ergoplatform.com/api/v0/assets/${selected}/issuingBox`)
			.then (res => res.json())
			.then (apiResponseToken => {
				return apiResponseToken || []
			})
		}
	}

	function toUtf8String(hex) {
		if(!hex){
			hex = ''
		}
	    var str = '';
	    for (var i = 0; i < hex.length; i += 2) {
	        str += String.fromCharCode(parseInt(hex.substr(i, 2), 16));
	    }
	    return str;
	}

	
</script>

<svelte:head>
	<script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN" crossorigin="anonymous"></script>
	<script src="popper.min.js"></script>
	<script src="bootstrap.js"></script>
	<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-eOJMYsd53ii+scO/bJGFsiCZc+5NDVN2yr8+0RDqr0Ql0h+rP48ckxlpbzKgwra6" crossorigin="anonymous">
	<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta3/dist/js/bootstrap.bundle.min.js" integrity="sha384-JEW9xMcG8R+pH31jmWH6WWP0WintQrMb4s7ZOdauHnUtxwoG2vI5DkLtS3qm9Ekf" crossorigin="anonymous"></script>
	<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.4.1/font/bootstrap-icons.css">
</svelte:head>

<main class=" bg-dark">
	<div class="row g-3 bg-dark px-3 py-3">
		<div class="col-sm-2 col-md-2">
			<a href="https://ergotokens.org" class="mb-1"><img src="ergo.png" alt="Logotype Ergo" width="100"></a>
		</div>
		<div class="col-7 col-md-8">
			<input class="form-control mx-2" on:input={handleInput} placeholder="Your id token or token name (5 letters min)" value={value}>
		</div>
		<div class="col-3 col-md-2">
			<a href="https://ergonfts.org" title="Ergo NFTs" class="btn bg-dark text-secondary border border-secondary ml-2">NFTs</a>
		</div>
	</div>
	
	<select bind:value={selected} class={claseSelect}>
	{#await response}
		<p>searching...</p>
	{:then response}
		{#if response.length > 0}
			<option value=0 selected>Select token</option>
			{cambiarDeColorSelect()}
			{#each response as token}
				<option value={token.id}>{token.name}</option>
			{/each}
		{:else}
			<span>No tokens</span>
		{/if}
	{:catch error}
		<p>✖️Something went wrong: {error.message}</p>
	{/await}
	</select>

	{#await informacionCompletaTokens}
		<span class="text-secondary">Loading...</span>
	{:then informacionCompletaTokens}
		{#if informacionCompletaTokens.length > 0}
			<ul class={claseListaGrupo}>
				<li class="list-group-item bg-dark text-light">Tokens info</li>
			{#each informacionCompletaTokens as ImagenToken}
				{#if ImagenToken.additionalRegisters.R9}
					<li class={claseLista}>
						{#if (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.gif') || (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.png') || (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.jpg') || (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.bmp') || (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == 'jpeg') || (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.svg') || (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.raw') || (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.nef')}
							<span><a href={toUtf8String(ImagenToken.additionalRegisters.R9).substr(2)} target="_blank" title={ImagenToken.assets[0].name}><img src={toUtf8String(ImagenToken.additionalRegisters.R9).substr(2)} class="img-thumbnail" width="200" alt={ImagenToken.assets[0].name} /></a></span>
						{:else if (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.mp3') || (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.ogg') || (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.wav') || (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.aac') || (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.wma') || (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == 'aiff')}
							<span>
								<!--<AudioPlayer
									src={toUtf8String(ImagenToken.additionalRegisters.R9).substr(2)}
									title={ImagenToken.assets[0].name}
								/> -->
								 <audio src={toUtf8String(ImagenToken.additionalRegisters.R9).substr(2)} title={ImagenToken.assets[0].name} controls></audio> 
							</span>
						{:else if (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.mp4') || (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.mov') || (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.3gp')}
							<span>
								<video src={toUtf8String(ImagenToken.additionalRegisters.R9).substr(2)} controls></video>
							</span>

						{/if}

					</li>
				{:else}
					<li class={claseLista}>
						<span class="text-danger">Aditional Register 9 Empty</span>
					</li>
				{/if}
				
				<li class={claseLista}>
					<span class="text-break"><small><strong>Id: </strong>{ImagenToken.id}</span>
				</li>
				<li class={claseLista}>
					<span class="text-break"><small><strong>Tx id: </strong>{ImagenToken.txId}</span>
				</li>
				<li class={claseLista}>
					<span class="text-break"><strong>Value: </strong>{ImagenToken.value}</span>
				</li>
				<li class={claseLista}>
					<span><small><strong>Creation Height: </strong>{ImagenToken.creationHeight}</span>
				</li>
				<li class={claseLista}>
					<span class="text-break"><small><strong>Ergo Tree: </strong>{ImagenToken.ergoTree}</span>
				</li>
				<li class={claseLista}>
					<span class="text-break"><small><strong>Address: </strong>{ImagenToken.address}</span>
				</li>
				<li class={claseLista}>
					<span class="text-break"><small><strong>Token Id: </strong>{ImagenToken.assets[0].tokenId}</span>
				</li>
				<li class={claseLista}>
					<span><small><strong>Amount: </strong>{ImagenToken.assets[0].amount}</span>
				</li>
				<li class={claseLista}>
					<span class="text-break"><small><strong>Name: </strong>{ImagenToken.assets[0].name}</span>
				</li>
				<li class={claseLista}>
					<span><small><strong>Decimals: </strong>{ImagenToken.assets[0].decimals}</span>
				</li>
				<li class={claseLista}>
					<span><small><strong>Type: </strong>{ImagenToken.assets[0].type}</span>
				</li>
				<li class="list-group-item bg-dark text-light">Token aditionals registers</li>
				<li class={claseLista}>
					<span class="text-break"><small><strong>R4 </strong>Token verbose name: 
						{#if ImagenToken.additionalRegisters.R4}
							{ImagenToken.additionalRegisters.R4}
						{:else}
							Empty
						{/if}
					<br>
					UTF-8 representation:<mark>{toUtf8String(ImagenToken.additionalRegisters.R4).substr(2)}</mark>
					</span>
				</li>
				<li class={claseLista}>
					<span class="text-break"><small><strong>R5 </strong> Token description:
						{#if ImagenToken.additionalRegisters.R5}
							{ImagenToken.additionalRegisters.R5}
						{:else}
							Empty
						{/if}
						<br>
						UTF-8 representation: <mark>{toUtf8String(ImagenToken.additionalRegisters.R5).substr(2)}</mark>
					</span>
				</li>
				<li class={claseLista}>	
					<span class="text-break"><small><strong>R6 </strong> Number of decimals:
						{#if ImagenToken.additionalRegisters.R6}
							{ImagenToken.additionalRegisters.R6}
						{:else}
							Empty
						{/if}
						<br>
						<mark>{ImagenToken.additionalRegisters.R6.length}</mark>
					</span>
				</li>
				<li class={claseLista}>	
					<span class="text-break"><small><strong>R7 </strong>NFT (Picture-0e020101 Audio-0e020102):
						{#if ImagenToken.additionalRegisters.R7}
							<br>
							<mark>{ImagenToken.additionalRegisters.R6}</mark>
						{:else}
							Empty
						{/if}
					</span>
				</li>
				<li class={claseLista}>
					<span class="text-break"><small><strong>R8 </strong>SHA256 hash of the picture: 
						{#if ImagenToken.additionalRegisters.R8}
							{ImagenToken.additionalRegisters.R8}
						{:else}
							Empty
						{/if}
					</span>

				</li>
				<li class={claseLista}>
					<span class="text-break"><small><strong>R9 </strong> Optional - link to the artwork (UTF-8 representation)
						{#if ImagenToken.additionalRegisters.R9}
							{ImagenToken.additionalRegisters.R9}
						{:else}
							Empty
						{/if}
						<br>
						<mark><a href={toUtf8String(ImagenToken.additionalRegisters.R9).substr(2)}>{toUtf8String(ImagenToken.additionalRegisters.R9).substr(2)}</a></mark>
					</span>
				</li>
			{/each}
			</ul>
		{/if}
	{:catch error}
		<p>✖️ Something went wrong: {error.message}</p>
	{/await}
</main>