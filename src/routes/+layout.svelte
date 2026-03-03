<script>
	import { supabase } from '$lib/supabaseClient';
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';

	let user = null;

	onMount(async () => {
		const { data } = await supabase.auth.getUser();
		user = data.user;

		if (!user) {
			goto('/login');
			return;
		}

		const { data: memberships } = await supabase
			.from('band_members')
			.select('*')
			.eq('email', user.email)
			.eq('active', true);

		if (!memberships || memberships.length === 0) {
			await supabase.auth.signOut();
			alert('Geen toegang tot deze app.');
			goto('/login');
			return;
		}

		for (const m of memberships) {
			if (!m.user_id) {
				await supabase.from('band_members').update({ user_id: user.id }).eq('id', m.id);
			}
		}
	});

	async function logout() {
		await supabase.auth.signOut();
		goto('/login');
	}
</script>

{#if user}
	<div
		style="background:#111; color:white; padding:8px 12px; display:flex; justify-content:space-between;"
	>
		<div>
			Ingelogd als: <strong>{user.email}</strong>
		</div>
		<button on:click={logout} style="background:#444; color:white; border:none; padding:4px 8px;">
			Logout
		</button>
	</div>
{/if}

<slot />
