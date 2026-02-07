<script>
	import { onMount } from 'svelte';
	import { supabase } from '$lib/supabaseClient';

	const BAND_ID = 1;
	let setlist = [];

	async function loadSetlist() {
		const { data, error } = await supabase
			.from('setlist_items')
			.select('id, position, songs(id, song, artist)')
			.eq('band_id', BAND_ID)
			.order('position', { ascending: true });

		if (error) {
			console.error('Load error:', error);
			return;
		}

		setlist = data ?? [];
	}

	// --- HIER GEBEURT HET ECHTE OMDRAAIEN IN DE DB ---
	async function swapPositions(itemA, itemB) {
		// 1) bewaar oude posities
		const posA = itemA.position;
		const posB = itemB.position;

		// 2) schrijf eerst A naar tijdelijke positie
		await supabase.from('setlist_items').update({ position: -999 }).eq('id', itemA.id);

		// 3) zet B op A z'n plek
		await supabase.from('setlist_items').update({ position: posA }).eq('id', itemB.id);

		// 4) zet A op B z'n plek
		await supabase.from('setlist_items').update({ position: posB }).eq('id', itemA.id);

		// 5) daarna opnieuw laden
		await loadSetlist();
	}

	async function moveUp(i) {
		if (i === 0) return;
		await swapPositions(setlist[i - 1], setlist[i]);
	}

	async function moveDown(i) {
		if (i === setlist.length - 1) return;
		await swapPositions(setlist[i], setlist[i + 1]);
	}

	async function removeFromSetlist(id) {
		await supabase.from('setlist_items').delete().eq('id', id);
		await loadSetlist();

		// netjes hernummeren
		const updates = setlist.map((item, i) => ({
			id: item.id,
			position: i + 1
		}));

		for (const u of updates) {
			await supabase.from('setlist_items').update({ position: u.position }).eq('id', u.id);
		}

		await loadSetlist();
	}

	onMount(loadSetlist);
</script>

<h1>Setlist</h1>

<ol>
	{#each setlist as item, i}
		<li>
			{item.songs.song} — {item.songs.artist}

			<button on:click={() => moveUp(i)}>↑</button>
			<button on:click={() => moveDown(i)}>↓</button>
			<button on:click={() => removeFromSetlist(item.id)}>x</button>
		</li>
	{/each}
</ol>
