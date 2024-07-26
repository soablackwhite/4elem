<script lang="ts">
	import '../app.postcss';
	import { initializeStores, Drawer, getDrawerStore } from '@skeletonlabs/skeleton';

	initializeStores();
	const drawerStore = getDrawerStore();
	// drawerStore.open();
	const drawerSettings: DrawerSettings = {
		id: 'example-3',
		// property overrides:
		bgDrawer: 'bg-surface-900 text-white',
		bgBackdrop: 'bg-gradient-to-tr from-white/30 via-violet-500/30 to-black/70',
		width: 'w-[300px] md:w-[600px]',
		padding: 'p-4',
		rounded: 'rounded-xl',
	};
	drawerStore.open(drawerSettings);
	drawerStore.close();

	function oppie(){
		drawerStore.open(drawerSettings);
	}

</script>
<svelte:head>
	<title> 4Elem | Battle of Elements </title>
</svelte:head>
<Drawer>
	<div class="instructions p-8">
		<h1 class="h1 text-primary-500"> INSTRUCTIONS</h1>
		<h2 class="h2 my-4"> Objective</h2>
		<p>Skully will hit you with a random spell picked from four elements. Guess the element of his card, show a hand sign to the camera, and when ready, press the attack button to start the duel.</p>
		<hr class="!border-t-2" />
		<h2 class="h2 my-4"> Table</h2>
		<div class="row">
			<div class="column">
				<div><span class="text-success-500">earth</span> &gt <span class="text-tertiary-500">water</span> </div>
				<div><span class="text-tertiary-500">water</span> &gt <span class="text-error-500">fire</span> </div>
			</div>
			<div class="column">
				<div><span class="text-error-500">fire</span> &gt <span class="text-primary-500">air</span></div>
				<div><span class="text-primary-500">air</span> &gt <span class="text-success-500">earth</span> </div>
			</div>
		</div>
		every other pairing results in a <span class="text-warning-500">draw</span>.
		<h2 class="h2 my-4"> Hints </h2>
		<ul>
			<li><i>use the information you have about the spells in Skully's hand to your advantage.</i></li> 
			<li><i>hint2: if Skully rans out of spells, he loses.</i></li>
		</ul>
		

	</div>
</Drawer>


<button type="button" on:click={oppie} class="moreinfo btn-icon btn-icon-l variant-filled">?</button>

<slot />

<style>
	*{
        font-family: 'Pixelify Sans', sans-serif;
	}
	ul{
		list-style-type:square;
		padding-left: 30px;
	}
	.qmark{
		width: 40px;
	}
	.row{
		display: flex;
		flex-direction: row;
		justify-content: space-evenly;
	}
	.column{
		display: flex;
		flex-direction: column;
	}
	.moreinfo{
		position: absolute;
		font-size: 2em;
		top: 8px;
		right: 8px;
	}
	.instructions{
		line-height: 2em;
		font-size: x-large;
	}
</style>

