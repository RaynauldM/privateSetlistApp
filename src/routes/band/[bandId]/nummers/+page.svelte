<script>
	import { onMount } from 'svelte';
	import { supabase } from '$lib/supabaseClient';
	import { page } from '$app/stores';

	let songs = [];
	let newSong = '';
	let newArtist = '';

	$: bandId = Number($page.params.bandId);

	// ----------------------
	// Songs laden
	// ----------------------

	async function loadSongs() {
		const { data, error } = await supabase
			.from('songs')
			.select('*')
			.eq('band_id', bandId)
			.order('song');

		if (error) {
			console.error('Load error:', error);
			return;
		}

		songs = (data ?? []).map((s) => ({
			...s,
			editing: false
		}));
	}

	// ----------------------
	// Song toevoegen
	// ----------------------

	async function addSong() {
		if (!newSong || !newArtist) return;

		const { error } = await supabase.from('songs').insert({
			song: newSong,
			artist: newArtist,
			band_id: bandId
		});

		if (error) {
			console.error('Insert error:', error);
			return;
		}

		newSong = '';
		newArtist = '';
		await loadSongs();
	}

	// ----------------------
	// Song verwijderen
	// ----------------------

	async function deleteSong(id) {
		await supabase.from('songs').delete().eq('id', id);
		await loadSongs();
	}

	// ----------------------
	// Editing (correct reactief)
	// ----------------------

	function startEdit(song) {
		songs = songs.map((s) =>
			s.id === song.id
				? {
						...s,
						editing: true,
						originalSong: s.song,
						originalArtist: s.artist
					}
				: s
		);
	}

	function cancelEdit(song) {
		songs = songs.map((s) =>
			s.id === song.id
				? {
						...s,
						song: s.originalSong,
						artist: s.originalArtist,
						editing: false
					}
				: s
		);
	}

	async function saveEdit(song) {
		const { error } = await supabase
			.from('songs')
			.update({
				song: song.song,
				artist: song.artist
			})
			.eq('id', song.id);

		if (error) {
			console.error('Update error:', error);
			return;
		}

		await loadSongs();
	}

	// ----------------------
	// Toevoegen aan setlist
	// ----------------------

	async function addToSetlist(songId) {
		// 1️⃣ setlist ophalen of aanmaken
		const { data: setlistData, error: setlistError } = await supabase
			.from('setlists')
			.select('id')
			.eq('band_id', bandId)
			.limit(1)
			.maybeSingle();

		if (setlistError) {
			console.error('Setlist fetch error:', setlistError);
			return;
		}

		let setlistId;

		if (setlistData) {
			setlistId = setlistData.id;
		} else {
			const { data: newSetlist, error: insertError } = await supabase
				.from('setlists')
				.insert({
					band_id: bandId,
					name: 'Actieve setlist'
				})
				.select()
				.single();

			if (insertError) {
				console.error('Setlist create error:', insertError);
				return;
			}

			setlistId = newSetlist.id;
		}

		// 2️⃣ positie bepalen
		const { data: last } = await supabase
			.from('setlist_items')
			.select('position')
			.eq('setlist_id', setlistId)
			.order('position', { ascending: false })
			.limit(1);

		const nextPos = last?.[0]?.position ? last[0].position + 1 : 1;

		// 3️⃣ insert
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
			{#if s.editing}
				<input bind:value={s.song} />
				<input bind:value={s.artist} />
				<button on:click={() => saveEdit(s)}>Opslaan</button>
				<button on:click={() => cancelEdit(s)}>Annuleer</button>
			{:else}
				{s.song} — {s.artist}
				<button on:click={() => startEdit(s)}>✏️</button>
				<button on:click={() => addToSetlist(s.id)}>→ setlist</button>
				<button on:click={() => deleteSong(s.id)}>🗑</button>
			{/if}
		</li>
	{/each}
</ul>
