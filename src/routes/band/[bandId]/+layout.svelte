<script>
	import { page } from '$app/stores';
	$: bandId = $page.params.bandId;
</script>

<nav>
	<a href={`/band/${bandId}/nummers`}>Alle nummers</a>
	<a href={`/band/${bandId}/setlist`}>Setlist</a>
</nav>

<slot />
