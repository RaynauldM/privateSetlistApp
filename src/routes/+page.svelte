<script>
	import { supabase } from '$lib/supabaseClient';
	import { onMount } from 'svelte';

	let bands = [];

	onMount(async () => {
		const {
			data: { user }
		} = await supabase.auth.getUser();

		const { data } = await supabase
			.from('band_members')
			.select('bands(id, name)')
			.eq('user_id', user.id)
			.eq('active', true);

		bands = data.map((d) => d.bands);
	});
</script>

<h1>Kies een band</h1>

{#each bands as band}
	<a href={`/band/${band.id}/nummers`}>
		<button>{band.name}</button>
	</a>
{/each}
