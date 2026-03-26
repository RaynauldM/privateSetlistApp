<script>
	import { supabase } from '$lib/supabaseClient';
	import { goto } from '$app/navigation';

	let email = '';
	let password = '';

	async function login() {
		const { error } = await supabase.auth.signInWithPassword({
			email,
			password
		});

		if (error) {
			console.error(error);
			alert('Login mislukt: ' + error.message);
			return;
		}

		// ✅ na login naar homepage (layout regelt rest)
		goto('/');
	}
</script>

<h1>Login</h1>

<div class="form">
	<input placeholder="Email" bind:value={email} />
	<input type="password" placeholder="Wachtwoord" bind:value={password} />

	<button type="button" on:click={login}> Login </button>
</div>

<hr />

<div class="guest">
	<h3>Of</h3>

	<a href="/jam">
		<button type="button"> 🎸 Gast van de Donderjam </button>
	</a>
</div>

<style>
	h1 {
		text-align: center;
		margin-bottom: 20px;
	}

	.form {
		display: flex;
		flex-direction: column;
		gap: 10px;
		max-width: 300px;
		margin: 0 auto;
	}

	input {
		padding: 8px;
		font-size: 1rem;
	}

	button {
		padding: 10px;
		font-size: 1rem;
		cursor: pointer;
	}

	.guest {
		text-align: center;
		margin-top: 20px;
	}
</style>
