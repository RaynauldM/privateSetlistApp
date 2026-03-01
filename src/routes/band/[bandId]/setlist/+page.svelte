<script>
	import { onMount } from 'svelte';
	import { supabase } from '$lib/supabaseClient';
	import { page } from '$app/stores';

	let setlist = [];
	let setlistId = null;

	// bandId uit URL halen
	$: bandId = Number($page.params.bandId);

	// Zorgt dat er altijd een setlist bestaat voor deze band
	async function ensureSetlist() {
		const { data, error } = await supabase
			.from('setlists')
			.select('id')
			.eq('band_id', bandId)
			.limit(1)
			.maybeSingle(); // voorkomt 406 error

		if (error) {
			console.error('Setlist fetch error:', error);
			return;
		}

		// Bestaat al
		if (data) {
			setlistId = data.id;
			return;
		}

		// Anders nieuwe maken
		const { data: newSetlist, error: insertError } = await supabase
			.from('setlists')
			.insert({
				band_id: bandId,
				name: 'Actieve setlist'
			})
			.select()
			.single();

		if (insertError) {
			console.error('Setlist creation error:', insertError);
			return;
		}

		setlistId = newSetlist.id;
	}

	async function loadSetlist() {
		await ensureSetlist();

		if (!setlistId) return;

		const { data, error } = await supabase
			.from('setlist_items')
			.select('id, position, songs(id, song, artist)')
			.eq('setlist_id', setlistId)
			.order('position', { ascending: true });

		if (error) {
			console.error('Load error:', error);
			return;
		}

		setlist = data ?? [];
	}

	async function swapPositions(itemA, itemB) {
		const posA = itemA.position;
		const posB = itemB.position;

		await supabase.from('setlist_items').update({ position: -999 }).eq('id', itemA.id);

		await supabase.from('setlist_items').update({ position: posA }).eq('id', itemB.id);

		await supabase.from('setlist_items').update({ position: posB }).eq('id', itemA.id);

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

	async function clearSetlist() {
		if (!confirm('Weet je zeker dat je de hele setlist wilt leegmaken?')) {
			return;
		}

		await supabase.from('setlist_items').delete().eq('setlist_id', setlistId);

		await loadSetlist();
	}

	onMount(loadSetlist);
</script>

<h1>Setlist</h1>
<button on:click={clearSetlist} style="margin-bottom: 1rem;"> Setlist leegmaken </button>
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
