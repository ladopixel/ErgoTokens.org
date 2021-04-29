<script>

function handleClick(){
	alert('a')
}

</script>

<svelte:head>
	
	<script src="popper.min.js"></script>
	<script src="bootstrap.js"></script>

	<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-eOJMYsd53ii+scO/bJGFsiCZc+5NDVN2yr8+0RDqr0Ql0h+rP48ckxlpbzKgwra6" crossorigin="anonymous">

	<script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN" crossorigin="anonymous"></script>

	<!-- JavaScript Bundle with Popper -->
	<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta3/dist/js/bootstrap.bundle.min.js" integrity="sha384-JEW9xMcG8R+pH31jmWH6WWP0WintQrMb4s7ZOdauHnUtxwoG2vI5DkLtS3qm9Ekf" crossorigin="anonymous"></script>
</svelte:head>

<main>
	<div class="row g-3 bg-dark px-3 py-3">
		<div class="col-sm-3 col-md-2">
			<a href="https://ergotokens.org" class="mb-1"><img src="ergo.png" width="100"></a>
			<span class="mx-2"> </span>
			<button class="btn bg-dark text-secondary border border-secondary">NFTs</button>
		</div>

		<div class="col-6 col-md-7">
			<input class="form-control mx-2" placeholder="Your wallet">
		</div>
		<div class="col-3 col-md-3">

			<button on:click={handleClick} class="btn bg-dark text-light border border-info ml-2">Accept</button>

            <button class="btn btn-secondary dropdown-toggle mr-2 bg-dark" href="#" id="dropdown01" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">Wallets</button>
            <div class="dropdown-menu" aria-labelledby="dropdown01" id="walletsSelect">
              <span class="dropdown-item">No wallets</span>
            </div>
          

		</div>
	</div>

</main>
