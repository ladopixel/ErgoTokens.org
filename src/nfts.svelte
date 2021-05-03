<script>
	let valorWallet = '9faPRhy5gSw4zv8bUNh3e6B4S9Y18NpwAqJtJuyu2hopPRPsj12'
	let arrayIds = []
	let objeto = {}
	let arrayDatos = []

	const listados = async() => {
		arrayDatos = []
		try {
			const res = await fetch(`https://api.ergoplatform.com/api/v0/addresses/${valorWallet}`)
			const data = await res.json()
			const arrayIds = data.transactions.confirmedTokensBalance.map(token => token.tokenId)
				for(let i = 0; i < arrayIds.length; i++){
					const res2 = await fetch(`https://api.ergoplatform.com/api/v0/assets/${arrayIds[i]}/issuingBox`)
					const data2 = await res2.json()
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
	<link href="https://fonts.googleapis.com/icon?family=Material+Icons"
      rel="stylesheet">
	<script src="popper.min.js"></script>
	<script src="bootstrap.js"></script>
	<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-eOJMYsd53ii+scO/bJGFsiCZc+5NDVN2yr8+0RDqr0Ql0h+rP48ckxlpbzKgwra6" crossorigin="anonymous">
	<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta3/dist/js/bootstrap.bundle.min.js" integrity="sha384-JEW9xMcG8R+pH31jmWH6WWP0WintQrMb4s7ZOdauHnUtxwoG2vI5DkLtS3qm9Ekf" crossorigin="anonymous"></script>
</svelte:head>

<!-- Cabecera -->
<main class="py-1 bg-dark">
	<div class="row g-3 bg-dark px-3 py-3">
		<div class="col-sm-3 col-md-2">
			<a href="https://ergotokens.org" class="mb-1"><img src="ergo.png" alt="Logotype Ergo" width="100"></a>
			<span class="mx-2"> </span>
			<button class="btn bg-dark text-secondary border border-secondary">NFTs</button>
		</div>

		<div class="col-6 col-md-7">
			<input class="form-control mx-2" placeholder="Your wallet" bind:value={valorWallet}>
		</div>
		<div class="col-3 col-md-3">
			<button on:click={listados} class="btn bg-dark text-light border border-info ml-2">Accept</button>
            <button class="btn btn-secondary dropdown-toggle mr-2 bg-dark" href="#" id="dropdown01" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">Wallets</button>
            <div class="dropdown-menu" aria-labelledby="dropdown01" id="walletsSelect">
              <span class="dropdown-item">No wallets</span>
            </div>
		</div>
	</div>	

<!-- Audio -->
<div class="mx-2 my-2 bg-light pb-1">
	<div class="bg-secondary">
		<button class="btn "><span id="listadoNFTsFavoritosAudio" class="material-icons mt-2 text-light">favorite</span></button>
		<span><strong>Your Audio NFTs </strong></span>
	</div>
  	<div class="row mx-2 my-2">
		{#each arrayDatos as datos}
			{#if datos.ext == '.mp3' || datos.ext == '.ogg' || datos.ext == '.wma' || datos.ext == '.aac' || datos.ext == 'aiff'}
				<div class="card mt-2 mx-1 cardColor" style="width: 18rem;">
					<div>
						<button class="btn"><span id="listadoNFTsFavoritosAudio" class="material-icons mt-2 text-light">favorite</span></button>
					</div>
						<audio src={datos.r9} class="card-img-top mb-3 " title={datos.name} controls></audio>
				</div>
			{/if}
		{/each}
	</div>
</div>

<!-- Picture -->
<div class="mx-2 my-2 bg-light pb-1">
	<div class="bg-secondary ">
		<button class="btn "><span id="listadoNFTsFavoritosAudio" class="material-icons mt-2 text-light">favorite</span></button>
		<span><strong>Your Picture NFTs </strong></span>
	</div>
  	<div class="row mx-2 my-2">
		{#each arrayDatos as datos}
			{#if datos.ext == '.png' || datos.ext == '.gif' || datos.ext == '.jpg'}
				<div class="card mt-2 mx-1 cardColor" style="width: 18rem;">
					<div>
						<button class="btn "><span id="listadoNFTsFavoritosPicture" class="material-icons mt-2 text-light">favorite</span></button>
					</div>
					<a href={datos.r9} title={datos.name}>
						<img src={datos.r9} class="card-img-top mb-3 imageBorder" alt={datos.name} width="200">
					</a>
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