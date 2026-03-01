<script>
	import { onMount } from 'svelte';
	import { supabase } from '$lib/supabaseClient';
	import { page } from '$app/stores';

	let songs = [];
	let newSong = '';
	let newArtist = '';

	$: bandId = Number($page.params.bandId);

	async function loadSongs() {
		const { data, error } = await supabase
			.from('songs')
			.select('*')
			.eq('band_id', bandId)
			.order('song');

		if (error) {
			console.error(error);
			return;
		}

		songs = data ?? [];
	}

	async function addSong() {
		if (!newSong || !newArtist) return;

		await supabase.from('songs').insert({
			song: newSong,
			artist: newArtist,
			band_id: bandId // 🔴 DIT IS CRUCIAAL
		});

		newSong = '';
		newArtist = '';

		await loadSongs();
	}

	async function deleteSong(id) {
		await supabase.from('songs').delete().eq('id', id);
		await loadSongs();
	}

	async function addToSetlist(songId) {
		// 1️⃣ haal de actieve setlist op van deze band
		const { data: setlistData, error: setlistError } = await supabase
			.from('setlists')
			.select('id')
			.eq('band_id', bandId)
			.limit(1)
			.single();

		if (setlistError) {
			console.error('Setlist fetch error:', setlistError);
			return;
		}

		const setlistId = setlistData.id;

		// 2️⃣ bepaal volgende positie
		const { data: last } = await supabase
			.from('setlist_items')
			.select('position')
			.eq('setlist_id', setlistId)
			.order('position', { ascending: false })
			.limit(1);

		const nextPos = last?.[0]?.position ? last[0].position + 1 : 1;

		// 3️⃣ insert correct met setlist_id
		const { error: insertError } = await supabase.from('setlist_items').insert({
			setlist_id: setlistId,
			song_id: songId,
			position: nextPos
		});

		if (insertError) {
			console.error('Insert error:', insertError);
			return;
		}
	}

	onMount(loadSongs);
</script>

<h1>Alle nummers</h1>

<div class="form">
	<input placeholder="Song" bind:value={newSong} />
	<input placeholder="Artist" bind:value={newArtist} />
	<button on:click={addSong}>Toevoegen</button>
</div>

<ul>
	{#each songs as s}
		<li>
			{s.song} — {s.artist}
			<button on:click={() => addToSetlist(s.id)}>→ setlist</button>
			<button on:click={() => deleteSong(s.id)}>🗑</button>
		</li>
	{/each}
</ul>
