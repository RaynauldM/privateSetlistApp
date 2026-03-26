<script>
	import { onMount } from 'svelte';
	import { supabase } from '$lib/supabaseClient';

	let setlist = [];

	const BAND_ID = 4; // 👈 pas aan als jouw id anders is

	async function loadSetlist() {
		// 1️⃣ haal setlist op
		const { data: setlistData } = await supabase
			.from('setlists')
			.select('id')
			.eq('band_id', BAND_ID)
			.limit(1)
			.maybeSingle();

		if (!setlistData) return;

		// 2️⃣ haal items op
		const { data } = await supabase
			.from('setlist_items')
			.select('position, songs(song, artist)')
			.eq('setlist_id', setlistData.id)
			.order('position');

		setlist = data ?? [];
	}

	onMount(loadSetlist);
</script>

<h1>🎸 Donderjam</h1>

<div class="topbar">
	<button on:click={loadSetlist}> 🔄 Vernieuwen </button>
</div>

{#if setlist.length === 0}
	<p class="empty">Nog geen nummers in de setlist</p>
{:else}
	<ol>
		{#each setlist as item, i}
			<li>
				<div class="index">{i + 1}</div>
				<div class="info">
					<div class="song">{item.songs.song}</div>
					<div class="artist">{item.songs.artist}</div>
				</div>
			</li>
		{/each}
	</ol>
{/if}

<style>
	:global(body) {
		background: #0f0f0f;
		color: white;
		font-family: system-ui, sans-serif;
		margin: 0;
		padding: 20px;
	}

	h1 {
		text-align: center;
		font-size: 2.2rem;
		margin-bottom: 20px;
		letter-spacing: 1px;
	}

	.topbar {
		display: flex;
		justify-content: center;
		margin-bottom: 25px;
	}

	.topbar button {
		background: #1f1f1f;
		color: white;
		border: 1px solid #333;
		padding: 12px 18px;
		font-size: 1rem;
		border-radius: 8px;
		cursor: pointer;
		transition: 0.2s;
	}

	.topbar button:hover {
		background: #333;
	}

	ol {
		list-style: none;
		padding: 0;
		margin: 0;
	}

	li {
		display: flex;
		align-items: center;
		gap: 15px;
		padding: 14px;
		margin-bottom: 12px;
		background: #1a1a1a;
		border-radius: 10px;
		border: 1px solid #2a2a2a;
	}

	.index {
		font-size: 1.4rem;
		font-weight: bold;
		opacity: 0.6;
		width: 30px;
	}

	.info {
		display: flex;
		flex-direction: column;
	}

	.song {
		font-size: 1.4rem;
		font-weight: bold;
		letter-spacing: 0.5px;
	}

	.artist {
		font-size: 0.9rem;
		opacity: 0.6;
		margin-top: 2px;
	}

	.empty {
		text-align: center;
		margin-top: 40px;
		opacity: 0.6;
	}
</style>
