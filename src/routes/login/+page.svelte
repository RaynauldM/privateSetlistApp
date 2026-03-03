<script>
	import { supabase } from '$lib/supabaseClient';

	let email = '';

	async function login() {
		const { error } = await supabase.auth.signInWithOtp({
			email
		});

		if (error) {
			alert('Login error');
			console.error(error);
		} else {
			alert('Check je email voor de magic link');
		}
	}
</script>

<h1>Login</h1>

<input placeholder="Email" bind:value={email} />
<button on:click={login}>Login</button>
