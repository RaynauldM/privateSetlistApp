<script>
	export let data;
</script>

<ul>
	{#each data.songs as song}
		<li>{song.song} by {song.artist}</li>
	{/each}
</ul>
