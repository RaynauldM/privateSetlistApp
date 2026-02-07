<script>
	import { page } from '$app/stores';
</script>

<nav class="header">
	<a href="/nummers" class:active={$page.url.pathname === '/nummers'}>Alle nummers</a>
	<a href="/setlist" class:active={$page.url.pathname === '/setlist'}>Setlist</a>
</nav>

<slot />

<style>
	.header {
		display: flex;
		gap: 1rem;
		padding: 12px;
		background: #111;
	}
	a {
		color: white;
		text-decoration: none;
	}
	.active {
		font-weight: bold;
		text-decoration: underline;
	}
</style>
