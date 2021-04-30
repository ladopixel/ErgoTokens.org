<script>

	let valorWallet = ''
	let arrayRegisters = []

	window.onload = function() {
  		verificaCookies()
	};

	function verificaCookies() {
	  if (document.cookie){
	    const cookieValor = document.cookie.replace(/(?:(?:^|.*;\s*)valorWallet\s*\=\s*([^;]*).*$)|^.*$/, '$1');

	    //alert(cookieValor)
	    //cargaDatos(cookieValor);
	  } else {
	    //alert('No hay cookie')
	  }
	}


	function mostrarListado(){
		fetch(`https://api.ergoplatform.com/api/v0/addresses/${valorWallet}`)
			.then (res => res.json())
			.then (apiResponse => {
				for (let i = 0; i<apiResponse.transactions.confirmedTokensBalance.length; i++){
					mostrarTokens(apiResponse.transactions.confirmedTokensBalance[i].tokenId,i)
				}
		})
	}


	function mostrarTokens(arrayTokens,i){
		fetch('https://api.ergoplatform.com/api/v0/assets/' + arrayTokens + '/issuingBox')
			.then (res => res.json())
			.then (apiResponse => {
				//alert(toUtf8String(apiResponse[0].additionalRegisters.R9).substr(2))
				if(apiResponse[0].additionalRegisters.R9){
					arrayRegisters[i] = apiResponse[0].additionalRegisters.R9
					//arrayRegisters = apiResponse
				}

			})
		}


	// https://api.ergoplatform.com/api/v0/addresses/${valorWallet}`
	// apiResponse.transactions.confirmedTokensBalance[0].tokenId
	
	//https://api.ergoplatform.com/api/v0/assets/' + idDelToken + '/issuingBox'
	//toUtf8String(apiResponse[0].additionalRegisters.R9).substr(2)




	function toUtf8String(hex) {
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
</svelte:head>
 
<main>
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

			<button on:click={mostrarListado} class="btn bg-dark text-light border border-info ml-2">Accept</button>

            <button class="btn btn-secondary dropdown-toggle mr-2 bg-dark" href="#" id="dropdown01" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">Wallets</button>
            <div class="dropdown-menu" aria-labelledby="dropdown01" id="walletsSelect">
              <span class="dropdown-item">No wallets</span>
            </div>
		</div>
	</div>	


	{#each arrayRegisters as nfts}
		<div class="card" style="width: 18rem;">
			{#if nfts}
				{#if toUtf8String(nfts).substr(2).slice(-4) == '.mp3'}
					<audio src={toUtf8String(nfts).substr(2)} class="card-img-top" alt="..." controls></audio>
					<div class="card-body">
				    	<h5 class="card-title"></h5>
				    	<button href="#" class="btn btn-danger"> </button>
				 	</div>
				{:else if toUtf8String(nfts).substr(2).slice(-4) == '.gif' || toUtf8String(nfts).substr(2).slice(-4) == '.png'}
					<img src={toUtf8String(nfts).substr(2)} class="card-img-top" alt="...">
				 	<div class="card-body">
				    	<h5 class="card-title"></h5>
				    	<button href="#" class="btn btn-danger"> </button>
				 	</div>
				 {/if}
			{/if}
		</div>
	{/each}


</main>
