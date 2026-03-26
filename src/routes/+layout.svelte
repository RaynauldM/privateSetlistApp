<script>
	import { supabase } from '$lib/supabaseClient';
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';

	let user = null;

	const publicRoutes = ['/login', '/jam'];

	onMount(async () => {
		// 🔑 huidige session ophalen
		const { data } = await supabase.auth.getSession();
		user = data.session?.user ?? null;

		await handleUser();

		// 🔄 luisteren naar login/logout
		supabase.auth.onAuthStateChange(async (_event, session) => {
			user = session?.user ?? null;
			await handleUser();
		});
	});

	async function handleUser() {
		// ❌ niet ingelogd → check of route public is
		if (!user && !publicRoutes.includes(window.location.pathname)) {
			goto('/login');
			return;
		}

		// ✅ ingelogd → check band toegang
		if (user) {
			const { data: memberships } = await supabase
				.from('band_members')
				.select('*')
				.eq('email', user.email)
				.eq('active', true);

			// ❌ geen toegang → eruit
			if (!memberships || memberships.length === 0) {
				await supabase.auth.signOut();
				goto('/login');
				return;
			}

			// 🔥 KOPPELING: user_id vullen indien leeg
			for (const m of memberships) {
				if (!m.user_id) {
					await supabase.from('band_members').update({ user_id: user.id }).eq('id', m.id);
				}
			}
		}
	}

	async function logout() {
		await supabase.auth.signOut();
		goto('/login');
	}
</script>

{#if user}
	<div class="header">
		<div class="left">
			<button on:click={() => goto('/')}> ← Kies band </button>
		</div>

		<div class="right">
			<span>{user.email}</span>
			<button on:click={logout}>Logout</button>
		</div>
	</div>
{/if}

<slot />

<style>
	.header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 10px 15px;
		background: #111;
		color: white;
		font-size: 0.9rem;
	}

	button {
		background: #444;
		color: white;
		border: none;
		padding: 6px 10px;
		cursor: pointer;
	}
</style>
