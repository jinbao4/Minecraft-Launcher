<script lang="ts">
	import { onMount } from 'svelte';
	import Layout from './Layout.svelte';
	import type { Account } from '$lib/types';
	import {
		tryAutoLogin,
		startLogin,
		logout,
		setupAuthListeners
	} from '$lib/helpers/auth';
	import {
		launchInstance
	} from '$lib/helpers/instances';
	import { setupInstallListeners } from '$lib/helpers/events';
	import Play from './pages/Play.svelte';
	import Install from './pages/Install.svelte';
	import Instances from './pages/Instances.svelte';

	let currentLocation = $state(window.location.pathname);
	let status = $state('Idle');
	let isLoggingIn = $state(false);
	let account = $state<Account | null>(null);

	onMount(async () => {
		const savedAccount = await tryAutoLogin();
		if (savedAccount) {
			status = `Refreshing session for ${savedAccount.name}...`;
			isLoggingIn = true;
			account = savedAccount;
			status = `Welcome back, ${account.name}`;
			isLoggingIn = false;
		}

		setupAuthListeners(
			(acc) => {
				account = acc;
				status = `Logged in as ${account.name}`;
				isLoggingIn = false;
			},
			(error) => {
				status = `Login Failed: ${error}`;
				isLoggingIn = false;
			}
		);

		setupInstallListeners(
			(installStatus) => {
				status = installStatus;
			},
			(error) => {
				status = `Error: ${error}`;
			}
		);
	});

	async function handleLogin(): Promise<void> {
		isLoggingIn = true;
		status = 'Opening Microsoft Login...';
		try {
			await startLogin();
		} catch (error) {
			console.error('Login error:', error);
			status = 'Failed to open login window';
			isLoggingIn = false;
		}
	}

	function handleLogout(): void {
		account = null;
		logout();
		status = 'Logged out.';
	}

	async function loginAndLaunch(name: string): Promise<void> {
		if (!account) {
			await handleLogin();
			return;
		}

		try {
			status = `Launching ${name}...`;
			await launchInstance(name, account);
		} catch (error) {
			console.error('Launch error:', error);
			status = `Launch Error: ${error}`;
		}
	}

	function navigate(location: string) {
		window.history.pushState({}, '', location);
		currentLocation = location;
	}

	$effect(() => {
		const handlePopState = () => {
			currentLocation = window.location.pathname;
		};
		window.addEventListener('popstate', handlePopState);
		return () => window.removeEventListener('popstate', handlePopState);
	});
</script>

<Layout {currentLocation} {navigate}>
	{#if currentLocation === '/' || currentLocation === ''}
		<Play
			{account}
			{status}
			{isLoggingIn}
			onLogin={handleLogin}
			onLogout={handleLogout}
			onLaunch={loginAndLaunch}
		/>
	{:else if currentLocation === '/install'}
		<Install {account} {status} {isLoggingIn} onLogin={handleLogin} onLogout={handleLogout} />
	{:else if currentLocation === '/instances'}
		<Instances
			{account}
			{status}
			{isLoggingIn}
			onLogin={handleLogin}
			onLogout={handleLogout}
			onLaunch={loginAndLaunch}
		/>
	{/if}
</Layout>
