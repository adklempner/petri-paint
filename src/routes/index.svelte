<script>
	import PetriLogo from "./PetriLogo.svelte";
</script>
<style>

	h1 {
		text-align: center;
		font-size: 2.8em;
		font-weight: 700;
		margin: 0 0 0.5em 0;
	}

	p {
		text-align: center;
		margin: 1em auto;
	}

</style>

<svelte:head>
	<title>petri-paint</title>
</svelte:head>

<div>
	<PetriLogo />

	<h1>petri-paint</h1> 

	<p><strong>Draw and Deploy Petri Nets</strong></p>

	<p><a href="editor">Try It ></a></p>
</div>