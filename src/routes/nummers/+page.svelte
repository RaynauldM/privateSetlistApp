<script>
	import { onMount } from 'svelte';
	import { supabase } from '$lib/supabaseClient';

	const BAND_ID = 1;

	let songs = [];
	let newSong = '';
	let newArtist = '';

	async function loadSongs() {
		const { data } = await supabase.from('songs').select('*').order('song');
		songs = data ?? [];
	}

	async function addSong() {
		if (!newSong || !newArtist) return;

		const { data } = await supabase
			.from('songs')
			.insert({ song: newSong, artist: newArtist })
			.select()
			.single();

		await supabase.from('band_songs').insert({
			band_id: BAND_ID,
			song_id: data.id
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
		const { data: last } = await supabase
			.from('setlist_items')
			.select('position')
			.eq('band_id', BAND_ID)
			.order('position', { ascending: false })
			.limit(1);

		const nextPos = last?.[0]?.position ? last[0].position + 1 : 1;

		await supabase.from('setlist_items').insert({
			band_id: BAND_ID,
			song_id: songId,
			position: nextPos
		});
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
