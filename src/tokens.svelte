<script>
	
	import {location, querystring} from 'svelte-spa-router'
	//https://github.com/meloalright/svelte-clipboard
	import Clipboard from "svelte-clipboard";

	let value = ''
	let response = []
	let informacionCompletaTokens = []
	let selected = ''

	let valorIdToken = ''

	let numWallets = 0
	let transitionParaWallet = ''
	let lastWalletToken = ''
	let actualWalletToken = ''

	let boxId = ''
	let idTransactions = ''
	let dateCreation = 0
	let dateCurrent = 0
	let ageToken = 0
	let fecha = 0

	let valorHTMLAscii = ''
	
	let metadata = ''
	let objetoTokenURL = {}
	let objetoMetadata = {}

	let objetoLastToken = {}

	let claseSelect = 'mb-2 mt-2 form-select form-select-sm '
	let claseNoImage = ' border border-secondary'
	let classCopyId = ' text-secondary'
	let classCopyWallet = ' text-secondary'

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
		// General 
		if ((selected != '') && (selected !=0)) {
			informacionCompletaTokens = fetch(`https://api.ergoplatform.com/api/v0/assets/${selected}/issuingBox`)
				.then (res => res.json())
				.then (apiResponseToken => {
					objetoTokenURL = {
							description: 'no metadata'
					}
					return apiResponseToken || []
			})
			// boxId para fecha y cartera
			fetch(`https://api.ergoplatform.com/api/v1/tokens/search?query=` + selected)
				.then(response => response.json())
				.then(consulta => {
					boxId = consulta.items[0].boxId
				})
				.catch(error => console.error(error));
			// Con el ID de la box averiguo el transactionsId
			fetch(`https://api.ergoplatform.com/api/v1/boxes/` + boxId)
				.then(response => response.json())
				.then(consulta => {
					idTransactions = consulta.transactionId
				})
				.catch(error => console.error(error));
			// Ascii
			fetch(`https://api.ergoplatform.com/api/v1/tokens/` + selected)
				.then(response => response.json())
				.then(consulta => {
					valorHTMLAscii = consulta.description
				})
				.catch(error => console.error(error));
		}		
	}

	// WALLETS Cada vez que se modifique select 
	$: {
		if ((selected != '') && (selected !=0)) {
			fetch(`https://api.ergoplatform.com/api/v1/boxes/byTokenId/${selected}`)
				.then (res => res.json())
				.then (apiResponseToken => {
					numWallets = apiResponseToken.total - 1
					// Rescato las fechas
					rescataFechas()
				fetch(`https://api.ergoplatform.com/api/v1/boxes/byTokenId/${selected}?offset=${numWallets}`)
					.then (res2 => res2.json())
					.then (apiResponseToken2 => {
						if (apiResponseToken2.items[0].address != undefined){
							actualWalletToken = apiResponseToken2.items[0].address
						}
					})
					.catch(error => console.error(error));
				})
				.catch(error => console.error(error));
		}
	}

	

	// Cargo metadatos cada vez que se carga algo del select
	$: {
		if ((selected != '') && (selected !=0)) {
			fetch(`https://api.ergoplatform.com/api/v0/assets/${selected}/issuingBox`)
				.then (res => res.json())
				.then (apiResponseToken => {
					// substr(3) para que cargue correctamente el metadata
					metadata = toUtf8String(apiResponseToken[0].additionalRegisters.R5).substr(3)
					if(isJson(metadata)){
						objetoTokenURL = {
							description: ''
						}
						objetoMetadata = JSON.parse(metadata)
						visualizoMetadata(objetoMetadata)
					}
				})
		}
	}

	

//////////////////////////////////////////////////////////////////////
//////////////////////////////////////////////////////////////////// Rescatar valor y mostrar token desde URL
//////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////////////////////////////////////

	valorIdToken = JSON.stringify($querystring)
	if (valorIdToken.substring(1, 6) == 'token'){
		valorIdToken = valorIdToken.substring(7, valorIdToken.length - 1)
		informacionCompletaTokens = fetch(`https://api.ergoplatform.com/api/v0/assets/${valorIdToken}/issuingBox`)
			.then (res => res.json())
			.then (apiResponseToken => {
				objetoTokenURL = {
					description: 'no metadata'
				}
				return apiResponseToken || []
		})
		// boxId para fecha y cartera
		fetch('https://api.ergoplatform.com/api/v1/tokens/search?query=' + valorIdToken)
			.then(response => response.json())
			.then(consulta => {
				boxId = consulta.items[0].boxId;
				sacarTransaccion(boxId)
			})
			.catch(error => console.error(error));
			
			function sacarTransaccion(box){
				// Con el ID de la box averiguo el transactionsId
				fetch('https://api.ergoplatform.com/api/v1/boxes/' + boxId)
					.then(response => response.json())
					.then(consulta => {
						idTransactions = consulta.transactionId;
						rescataFechasURL(idTransactions)
					})
					.catch(error => console.error(error));
			}
			
			// Ascii
		fetch('https://api.ergoplatform.com/api/v1/tokens/' + valorIdToken)
			.then(response => response.json())
			.then(consulta => {
				valorHTMLAscii = consulta.description
			})
			.catch(error => console.error(error));
		
			// WALLETS
		fetch(`https://api.ergoplatform.com/api/v1/boxes/byTokenId/${valorIdToken}`)
			.then (res => res.json())
			.then (apiResponseToken => {
				numWallets = apiResponseToken.total - 1
				// Rescato las fechas
				rescataFechas()
			fetch(`https://api.ergoplatform.com/api/v1/boxes/byTokenId/${valorIdToken}?offset=${numWallets}`)
				.then (res2 => res2.json())
				.then (apiResponseToken2 => {
					if (apiResponseToken2.items[0].address != undefined){
						actualWalletToken = apiResponseToken2.items[0].address
					}
				})
				.catch(error => console.error(error));
			})
			.catch(error => console.error(error));
	}

	function rescataFechasURL(id){
		fetch(`https://api.ergoplatform.com/api/v1/transactions/${id}`)
			.then (res3 => res3.json())
			.then (apiResponseToken3 => {
					// Rescato wallet de creación
					transitionParaWallet = apiResponseToken3.inputs[0].outputTransactionId
					fetch(`https://api.ergoplatform.com/api/v1/transactions/${transitionParaWallet}`)
						.then (res4 => res4.json())
						.then (apiResponseToken4 => {
							dateCreation = apiResponseToken4.timestamp
							// Creation Date
							fecha = new Date(dateCreation); 
							dateCreation = fecha.getDate()+
							"/"+(fecha.getMonth()+1)+
							"/"+fecha.getFullYear()
							// Actual Date
							dateCurrent = new Date().getTime()
							ageToken = restaFechas(dateCurrent, apiResponseToken4.timestamp)
							
							// Billetera de acuñacion
							lastWalletToken = apiResponseToken4.inputs[0].address
					})
					.catch(error => console.error(error));
					
			})
			.catch(error => console.error(error));
	}












	// Rescatar valor Metadata y mostrar token desde URL
	valorIdToken = JSON.stringify($querystring)
	if (valorIdToken.substring(1, 6) == 'token'){
		valorIdToken = valorIdToken.substring(7, valorIdToken.length - 1)
		fetch(`https://api.ergoplatform.com/api/v0/assets/${valorIdToken}/issuingBox`)
			.then (res => res.json())
			.then (apiResponseToken => {
				metadata = toUtf8String(apiResponseToken[0].additionalRegisters.R5).substr(3)
				if(isJson(metadata)){
					objetoTokenURL = {
						description: ''
					}
					objetoMetadata = JSON.parse(metadata)
					visualizoMetadata(objetoMetadata)
				}
			})
	}	

	function restaFechas(timestamp1, timestamp2) {
		var difference = timestamp1 - timestamp2;
		var daysDifference = Math.floor(difference/1000/60/60/24);
		return daysDifference;
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

	function resolveIpfs(url) {
		const ipfsPrefix = 'ipfs://'
		if (!url.startsWith(ipfsPrefix)) return url
		else return url.replace(ipfsPrefix, 'https://cloudflare-ipfs.com/ipfs/')
	}

	function isJson(str) {
		try {
			JSON.parse(str);
		} catch (e) {
			return false;
		}
		return true;
	}

	function visualizoMetadata(obj){
		for(var key in obj){
			if(!obj.hasOwnProperty(key)) continue;
				if(typeof obj[key] !== 'object') {
					if(key == 'image'){
						objetoTokenURL.description = objetoTokenURL.description + '<strong>' + letraMayuscula(key) + '</strong>: ' + '<a href="' + obj[key] + '">' + obj[key].substring(8, 30) + '...</a><br>'
					}else if(!Array.isArray(obj)){
						objetoTokenURL.description = objetoTokenURL.description + '<strong>' + letraMayuscula(key) + '</strong>: ' + obj[key] + '<br>'
					}
				} else {
					visualizoMetadata(obj[key])
				}
				if (Array.isArray(obj[key])){
					objetoTokenURL.description = objetoTokenURL.description + '<strong> ' + letraMayuscula(key) + ': </strong>' + obj[key] + '<br>'
				}
		}
	}

	function letraMayuscula(texto) {
		return texto.charAt(0).toUpperCase() + texto.slice(1);
	}

	// Date Age
	function rescataFechas(){
		fetch(`https://api.ergoplatform.com/api/v1/transactions/${idTransactions}`)
			.then (res3 => res3.json())
			.then (apiResponseToken3 => {
					// Rescato wallet de creación
					transitionParaWallet = apiResponseToken3.inputs[0].outputTransactionId
					fetch(`https://api.ergoplatform.com/api/v1/transactions/${transitionParaWallet}`)
						.then (res4 => res4.json())
						.then (apiResponseToken4 => {
							dateCreation = apiResponseToken4.timestamp
							// Creation Date
							fecha = new Date(dateCreation); 
							dateCreation = fecha.getDate()+
							"/"+(fecha.getMonth()+1)+
							"/"+fecha.getFullYear()
							// Actual Date
							dateCurrent = new Date().getTime()
							ageToken = restaFechas(dateCurrent, apiResponseToken4.timestamp)
							
							// Billetera de acuñacion
							lastWalletToken = apiResponseToken4.inputs[0].address
					})
					.catch(error => console.error(error));
					
			})
			.catch(error => console.error(error));
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

	<!-- LastToken-->	
	<!-- <button on:click|once={lastToken} class="btn btn-sm border border-secondary text-secondary"><i class="bi bi-chevron-bar-right"></i> Last token minted in Ergo</button> -->

	{#await informacionCompletaTokens}
		<span class="text-secondary">Loading...</span>
	{:then informacionCompletaTokens}
		{#if informacionCompletaTokens.length > 0}

			<center><h4 class="text-secondary center">Token info</h4></center>
			{#each informacionCompletaTokens as ImagenToken}

			<div class="container">
				<div class="card px-3 py-3" >
					<div class="row no-gutters">
						<div class="col-lg-3">							
							{#if (ImagenToken.additionalRegisters.R7 == '0e020101')}
								<img src={resolveIpfs(toUtf8String(ImagenToken.additionalRegisters.R9).substr(2))} class="card-img-top rounded" style="width: 16rem;" alt={ImagenToken.assets[0].name} />
							{:else if ImagenToken.additionalRegisters.R7 == '0e0430313031'}
								<center>
									<div class="mark mt-3 py-3 {claseNoImage}">
										Token minted with other standards. <br>
										<h4>Look at its Descriptions.</h4>
									</div>
								</center>
							{:else if ImagenToken.additionalRegisters.R7 == '0e020102'}
								<span>
									<center><audio src={resolveIpfs(toUtf8String(ImagenToken.additionalRegisters.R9).substr(2))} style="width: 12rem;" title={ImagenToken.assets[0].name} controls></audio> </center>
								</span>
							{:else if (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.mp4') || (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.mov') || (toUtf8String(ImagenToken.additionalRegisters.R9).slice(-4) == '.3gp')}
								<span>
									<video src={resolveIpfs(toUtf8String(ImagenToken.additionalRegisters.R9).substr(2))} controls></video>
								</span>
							{:else}
								<center>
									<div class="mark mt-3 py-3 {claseNoImage}">No image-audio token</div>
								</center>
							{/if}
						</div>
						<div class="col-lg-9" >
							<div class="card-body bg-light rounded px-3 py-2">
								<!-- Name -->
								<h5 class="card-title text-dark">
									<!-- Twitter button -->
									<a class="btn text-dark" href="https://www.google.com/searchbyimage?site=search&sa=X&image_url={resolveIpfs(toUtf8String(ImagenToken.additionalRegisters.R9).substr(2))}" target="_blank"><i class="bi bi-binoculars-fill"></i></a>
									<a class="btn text-info" href="https://twitter.com/intent/tweet?text=Enjoy%20this%20token%20&url=https://ergotokens.org%2F%23%2f?token={ImagenToken.assets[0].tokenId}%20created%20in%20@ergoplatformorg%20♥%20$ERG&hashtags=ErgoNFT,Ergo2Top10" target="_blank"><i class="bi bi-twitter"></i></a>
									{ImagenToken.assets[0].name}
								</h5>

								<!-- ID -->
								<div class="bg-white small mt-1 mb-1 px-2 py-1 rounded">
									<small>
										{#if ImagenToken.additionalRegisters.R7 == '0e020101'}
											<i class="bi bi-file-image"></i>
										{:else if ImagenToken.additionalRegisters.R7 == '0e020102'}
											<i class="bi bi-file-earmark-music"></i>
										{:else}
											<i class="bi bi-file-earmark-x"></i>
										{/if}
										<strong>ID </strong>
										<span class="text-secondary">
											{ImagenToken.assets[0].tokenId}
											<Clipboard 
												text={ImagenToken.assets[0].tokenId}
												let:copy
												on:copy={() => {
													classCopyId = ' text-info'
												}}>
												
												<button on:click={copy} class="btn btn-sm border border-white bg-white {classCopyId}">
													<i class="bi bi-files"></i>
												</button>
											</Clipboard>
										</span>
									</small>
								</div>

								<!-- dateCreation -->
								<div class="bg-white small mt-1 mb-1 px-2 py-1 rounded">
									<small>
										<i class="bi bi-calendar-event"></i>
										<strong>Date of birth:   </strong>
										<span class="text-secondary">
											{dateCreation}
										</span>
										<strong>Age: </strong>
											{ageToken}
										<span> days</span>
									</small>
								</div>

								<!-- SHA -->
								{#if ImagenToken.additionalRegisters.R8}
									<div class="bg-white small mt-1 mb-1 px-2 py-1 rounded">
										<small>
											<i class="bi bi-patch-check"></i>
											<strong>SHA256 </strong>
											<span class="text-secondary">
												{ImagenToken.additionalRegisters.R8.substr(4)}
											</span>
										</small>
									</div>
								{/if}

								<!-- Creation Height -->
								<div class="bg-white small mt-1 mb-1 px-2 py-1 rounded">
									<small>
										<i class="bi bi-bricks"></i>
										<strong>Creation Height </strong>
										<span class="text-secondary">
											{ImagenToken.creationHeight}
										</span>
									</small>
								</div>
									
								<!-- Minting Address -->
								<div class="bg-white small mt-1 mb-1 px-2 py-1 rounded">
									<span class="text-break">
										<small>
											<i class="bi bi-wallet2"></i>
											<strong>Minting address </strong>
											<span class="text-secondary">
												{lastWalletToken}
											</span>
										</small>
									</span>
								</div>

								<!-- Address -->
								<div class="bg-white small mt-1 mb-1 px-2 py-1 rounded">
									<span class="text-break">
										<small>
											<i class="bi bi-wallet"></i>
											<strong>Current address </strong>
											<span class="text-secondary">
												{actualWalletToken}
												<!-- <Clipboard
													text={actualWalletToken}
													let:copy
													on:copy={() => {
														classCopyWallet = ' text-info'
													}}>

													<button on:click={copy} class="btn btn-sm border border-white bg-white {classCopyWallet}">
														<i class="bi bi-files"></i>
													</button>
												</Clipboard> -->
											</span>
										</small>
									</span>
								</div>

								<!-- Amount -->
								<div class="bg-white small mt-1 mb-1 px-2 py-1 rounded">
									<span class="text-break">
										<small>
											<i class="bi bi-stack"></i>
											<strong>Amount </strong>
											<span class="text-secondary">
												{ImagenToken.assets[0].amount}
											</span>
										</small>
									</span>
								</div>

								<!-- Decimals -->
								<div class="bg-white small mt-1 mb-1 px-2 py-1 rounded">
									<span class="text-break">
										<small>
											<i class="bi bi-file-binary"></i>
											<strong>Decimals </strong>
											<span class="text-secondary">
												{ImagenToken.assets[0].decimals}
											</span>
										</small>
									</span>
								</div>

								<!-- Type -->
								<div class="bg-white small mt-1 mb-1 px-2 py-1 rounded">
									<span class="text-break">
										<small>
											<i class="bi bi-file-earmark-spreadsheet"></i>
											<strong>Type </strong>
											<span class="text-secondary">
												{ImagenToken.assets[0].type}
											</span>
										</small>
									</span>
								</div>
							</div>
						</div>
						
						<!-- Column 2 -->
						<div class="card-body bg-light rounded px-3 py-2 mx-2 my-2">
							<!-- Description -->
							<h6 class="mt-2">Descriptions</h6>
							{#if ImagenToken.additionalRegisters.R5}
								<div class="accordion accordion-flush" id="accordionFlushExample">
									<div class="accordion-item">
										<h2 class="accordion-header" id="flush-headingZero">
											<button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#flush-collapseZero" aria-expanded="false" aria-controls="flush-collapseZero">
												<small>Description decode</small>
											</button>
										</h2>
										<div id="flush-collapseZero" class="accordion-collapse collapse" aria-labelledby="flush-headingZero" data-bs-parent="#accordionFlushExample">
											<div class="accordion-body small text-secondary">
												{toUtf8String(ImagenToken.additionalRegisters.R5).substr(2)}
											</div>
										</div>
									</div>
									<div class="accordion-item">
										<h2 class="accordion-header" id="flush-headingOne">
											<button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#flush-collapseOne" aria-expanded="false" aria-controls="flush-collapseOne">
												<small>Description code</small>
											</button>
										</h2>
										<div id="flush-collapseOne" class="accordion-collapse collapse" aria-labelledby="flush-headingOne" data-bs-parent="#accordionFlushExample">
											<div class="accordion-body small text-secondary">
												{ImagenToken.additionalRegisters.R5}
											</div>
										</div>
									</div>
									<div class="accordion-item">
									<h2 class="accordion-header" id="flush-headingThree">
										<button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#flush-collapseThree" aria-expanded="false" aria-controls="flush-collapseThree">
											<small>Description metadata</small>
										</button>
									</h2>
									<div id="flush-collapseThree" class="accordion-collapse collapse" aria-labelledby="flush-headingThree" data-bs-parent="#accordionFlushExample">
										<div class="accordion-body small text-secondary">
											{@html decodeURIComponent(objetoTokenURL.description)}
										</div>
									</div>
									</div>
									<div class="accordion-item">
										<h2 class="accordion-header" id="flush-headingFour">
										<button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#flush-collapseFour" aria-expanded="false" aria-controls="flush-collapseFour">
											<small>Description Ascii</small>
										</button>
										</h2>
										<div id="flush-collapseFour" class="accordion-collapse collapse" aria-labelledby="flush-headingFour" data-bs-parent="#accordionFlushExample">
										<div class="accordion-body small text-dark">
											<pre class="maritsaart">
												<span>{valorHTMLAscii}</span>
											</pre>
										</div>
										</div>
									</div>
								</div>
							{:else}
								<center>
									<span class="mark">No description</span>
								</center>
							{/if}
						<!-- </div> -->

						<!-- <div class="card-body bg-light rounded px-3 py-2 mt-3"> -->
							<h6 class="mt-3">Additional registers</h6>
						<div class="accordion accordion-flush" id="accordionFlushExample">
							<div class="accordion-item">
								<h2 class="accordion-header" id="flush-headingR4">
									<button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#flush-collapseR4" aria-expanded="false" aria-controls="flush-collapseR4">
										<small>R4 - Token verbose name UTF-8 representation</small>
									</button>
								</h2>
							<div id="flush-collapseR4" class="accordion-collapse collapse" aria-labelledby="flush-headingR4" data-bs-parent="#accordionFlushExample">
								<div class="accordion-body">
									<span class="text-break"><small><strong>Token verbose name - UTF-8 representation</strong><br>
										{#if ImagenToken.additionalRegisters.R4}
											<mark>{toUtf8String(ImagenToken.additionalRegisters.R4).substr(2)}</mark>
										{:else}
											<small>Empty</small>
										{/if}
									</span>
								</div>
							</div>
						</div>
						<div class="accordion-item">
							<h2 class="accordion-header" id="flush-headingR5">
								<button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#flush-collapseR5" aria-expanded="false" aria-controls="flush-collapseR5">
									<small>R5 - Token description</small>
								</button>
							</h2>
							<div id="flush-collapseR5" class="accordion-collapse collapse" aria-labelledby="flush-headingR5" data-bs-parent="#accordionFlushExample">
								<div class="accordion-body">
									<span class="text-break"><small><strong>Token description</strong><br>
										{#if ImagenToken.additionalRegisters.R5}
											<small><mark>{toUtf8String(ImagenToken.additionalRegisters.R5).substr(2)}</mark></small>
										{:else}
											<small>Empty</small>
										{/if}
									</span>
								</div>
							</div>
						</div>
						<div class="accordion-item">
							<h2 class="accordion-header" id="flush-headingR6">
								<button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#flush-collapseR6" aria-expanded="false" aria-controls="flush-collapseR6">
									<small>R6 - Number of decimals</small>
								</button>
							</h2>
							<div id="flush-collapseR6" class="accordion-collapse collapse" aria-labelledby="flush-headingR6" data-bs-parent="#accordionFlushExample">
								<div class="accordion-body">
									<span class="text-break"><small><strong>Number of decimals</strong><br>
										{#if ImagenToken.additionalRegisters.R6}
											<small><mark>{toUtf8String(ImagenToken.additionalRegisters.R6).substr(2)}</mark></small>
										{:else}
											<small>Empty</small>
										{/if}
									</span>
								</div>
							</div>
						</div>
						<div class="accordion-item">
							<h2 class="accordion-header" id="flush-headingR7">
							<button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#flush-collapseR7" aria-expanded="false" aria-controls="flush-collapseR7">
								<small>R7 - NFT (Picture-0e020101 Audio-0e020102)</small>
							</button>
							</h2>
							<div id="flush-collapseR7" class="accordion-collapse collapse" aria-labelledby="flush-headingR7" data-bs-parent="#accordionFlushExample">
								<div class="accordion-body">
									<span class="text-break"><small><strong>NFT (Picture-0e020101 Audio-0e020102)</strong><br>
										{#if ImagenToken.additionalRegisters.R7}
											<small><mark>{ImagenToken.additionalRegisters.R7}</mark></small>
										{:else}
											<small>Empty</small>
										{/if}
									</span>
								</div>
							</div>
						</div>
						<div class="accordion-item">
							<h2 class="accordion-header" id="flush-headingR8">
							<button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#flush-collapseR8" aria-expanded="false" aria-controls="flush-collapseR8">
								<small>R8 - SHA256 Hash of the file</small>
							</button>
							</h2>
							<div id="flush-collapseR8" class="accordion-collapse collapse" aria-labelledby="flush-headingR8" data-bs-parent="#accordionFlushExample">
								<div class="accordion-body">
									<span class="text-break"><small><strong>SHA256 Hash of the file</strong><br>
										{#if ImagenToken.additionalRegisters.R8}
											<small><mark>{(ImagenToken.additionalRegisters.R8).substr(4)}</mark></small>
										{:else}
											<small>Empty</small>
										{/if}
									</span>
								</div>
							</div>
						</div>
						<div class="accordion-item">
							<h2 class="accordion-header" id="flush-headingR9">
							<button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#flush-collapseR9" aria-expanded="false" aria-controls="flush-collapseR9">
								<small>R9 (Optional) - Link to the artwork (UTF-8 representation)</small>
							</button>
							</h2>
							<div id="flush-collapseR9" class="accordion-collapse collapse" aria-labelledby="flush-headingR9" data-bs-parent="#accordionFlushExample">
								<div class="accordion-body">
									<span class="text-break"><small><strong>Optional - link to the artwork (UTF-8 representation)</strong><br>
										{#if ImagenToken.additionalRegisters.R9}
											<small><mark><a href={resolveIpfs(toUtf8String(ImagenToken.additionalRegisters.R9).substr(2))} target="_blank">{resolveIpfs(toUtf8String(ImagenToken.additionalRegisters.R9).substr(2))}</a></mark></small>
										{:else}
											<small>Empty</small>
										{/if}
									</span>
								</div>
							</div>
						</div>
					</div>
					</div>
					<!-- Link NFTs -->
					<div class="my-2">
						<a href="https://ergonfts.org/#/?token={ImagenToken.assets[0].tokenId}" class="btn btn-sm bg-white text-secondary border border-info small" target="_blank">View in ErgoNFTs.org</a>
						<a href="https://explorer.ergoplatform.com/en/transactions/{ImagenToken.txId}" class="btn btn-sm bg-white text-secondary border border-info small" target="_blank">See transition</a>
					</div>
				</div>
			</div>
			</div>
			{/each}
		{/if}
	{:catch error}
		<p>✖️ Something went wrong: {error.message}</p>
	{/await}
</main>