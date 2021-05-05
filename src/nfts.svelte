<script>

	let valorWallet = '9faPRhy5gSw4zv8bUNh3e6B4S9Y18NpwAqJtJuyu2hopPRPsj12'
	let claseGuardarLocalstorage = ''
	const onFocus = () => valorWallet = '';

	let arrayIds = []
	let objeto = {}
	let arrayDatos = []

	let arrayWallets = []
	let infoWallet = {id: '', address: ''}
	let selected = ''

	let arrayFavorites = []
	let infoFavorite = {id: '', status: ''}
	let claseFavorite = ''

	if (localStorage.getItem("arrayWallets")){
		arrayWallets = JSON.parse(localStorage.getItem("arrayWallets"))
	}
	$: localStorage.setItem("arrayWallets", JSON.stringify(arrayWallets))

	function guardarLocalstorage(){
		var mensaje = confirm(`Do you want to save this wallet in your browser?\n${valorWallet} \n\n This way you don't have to reintroduce it :)`);
		if (mensaje){
			infoWallet.id = Date.now()
			infoWallet.address = valorWallet
			arrayWallets = [...arrayWallets, infoWallet]
			infoWallet = {id: '', address: ''}
		}
	}

	if (localStorage.getItem("arrayFavorites")){
		arrayFavorites = JSON.parse(localStorage.getItem("arrayFavorites"))
	}
	$: localStorage.setItem("arrayFavorites", JSON.stringify(arrayFavorites))

	function addFavorite(idNft){
		let isFavorite = false
		for(let i=0; i < arrayFavorites.length; i++){
			if(idNft[0] == arrayFavorites[i].id){
				arrayFavorites = arrayFavorites.filter(item => item.id !== idNft[0])
				isFavorite = true
			}
		}
		if (isFavorite == false){
			infoFavorite.id = idNft[0]
			infoFavorite.status = false
			arrayFavorites = [...arrayFavorites, infoFavorite]
			infoFavorite = {id: '', status: false}
		}
	}

	$: {
		if ((selected != '') && (selected !=0)) {
			valorWallet = selected
			listados()
		}
	}

	const listados = async() => {
		arrayDatos = []
		try {
			const res = await fetch(`https://api.ergoplatform.com/api/v0/addresses/${valorWallet}`)
			const data = await res.json()
			const arrayIds = data.transactions.confirmedTokensBalance.map(token => token.tokenId)
				for(let i = 0; i < arrayIds.length; i++){
					const res2 = await fetch(`https://api.ergoplatform.com/api/v0/assets/${arrayIds[i]}/issuingBox`)
					const data2 = await res2.json()
					claseGuardarLocalstorage = claseGuardarLocalstorage + ' bg-info'
					objeto = {
						id: data2.map(token => token.assets[0].tokenId),
						name: data2.map(token => token.assets[0].name),
						r9: data2.map(token => toUtf8String(token.additionalRegisters.R9).substr(2)),
						ext: data2.map(token => toUtf8String(token.additionalRegisters.R9).substr(2).slice(-4))
					}
					arrayDatos[i] = objeto
				}
		} catch (error) {
			console.log(error)
		}
	}

	function listFavoritePicture() {
			for(let i = 0; i < arrayFavorites.length; i++){
				
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

<!-- Cabecera -->
<main class="py-1 bg-dark">
	<div class="row g-3 bg-dark px-3 py-3">
		<div class="col-sm-3 col-md-3">
			<a href="https://ergotokens.org" class="mb-1"><img src="ergo.png" alt="Logotype Ergo" width="100"></a>
			<span class="mx-2"> </span>
			<button class="btn bg-dark text-secondary border border-secondary">Tokens</button>
		</div>
		<div class="col-6 col-md-6">
			<div class="input-group mb-3 bg-dark">
				<button on:click={guardarLocalstorage} class="input-group-text dropdown {claseGuardarLocalstorage}" id="basic-addon1"><i class="bi bi-patch-plus"></i></button>
				<input on:focus={onFocus} bind:value={valorWallet} class="form-control" placeholder="Your wallet"  aria-label="Username" aria-describedby="basic-addon1">
			</div>
		</div>
		<div class="col-3 col-md-3">
			<button on:click={listados} class="btn bg-dark text-light border border-secondary ml-2">Accept</button>
			<select bind:value={selected} class="btn btn-secondary bg-dark">
				<option value=0 class="dropdown-item" selected>Select wallet</option>
				{#each arrayWallets as wallet}
					<option value={wallet.address} class="dropdown-item">{wallet.address.substring(0, 7)}...{wallet.address.slice(-7)}</option>
				{/each}
			</select>

		</div>
	</div>	

	
	<!-- Picture -->
	<div class="mx-2 my-2 bg-light pb-1">
		<div class="bg-secondary ">
			<button class="btn " on:click={listFavoritePicture}><i class="bi bi-card-image" title="List favorite"></i></button>
			<span><strong>Your Picture NFTs </strong></span>
		</div>
		<div class="row mx-2 my-2">
			{#each arrayDatos as datos}
				{#if datos.ext == '.png' || datos.ext == '.gif' || datos.ext == '.jpg'}
					<div class="card mt-2 mx-1 cardColor" style="width: 18rem;">
						<div>
							<button class="btn {claseFavorite}" on:click={addFavorite(datos.id)}><i class="bi bi-heart-fill text-light" title="Add favorite"></i></button>
						</div>
						<a href={datos.r9} title={datos.name}>
							<img src={datos.r9} class="card-img-top mb-3 imageBorder" alt={datos.name} width="200">
						</a>
					</div>
				{/if}
			{/each}
		</div>
	</div>

	<!-- Audio -->
	<div class="mx-2 my-2 bg-light pb-1">
		<div class="bg-secondary">
			<button class="btn"><i class="bi bi-caret-right-square-fill" title="List favorite"></i></button>
			<span><strong>Your Audio NFTs </strong></span>
			
		</div>
		<div class="row mx-2 my-2">
			{#each arrayDatos as datos}
				{#if datos.ext == '.mp3' || datos.ext == '.ogg' || datos.ext == '.wma' || datos.ext == '.aac' || datos.ext == 'aiff'}
					<div class="card mt-2 mx-1 cardColor" style="width: 18rem;">
						<div>
							<button class="btn {claseFavorite}" on:click={addFavorite(datos.id)}><i class="bi bi-heart-fill text-light" title="Add favorite"></i></button>
						</div>
							<audio src={datos.r9} class="card-img-top mb-3 " title={datos.name} controls></audio>
					</div>
				{/if}
			{/each}
		</div>
	</div>
</main>


<style>
	.cardColor{
		background-color: rgb(190, 190, 190);
		border: 0px;
	}
	.imageBorder{
		border: 5px solid #ffffff;
	}
</style>