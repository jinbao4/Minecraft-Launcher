<script lang="ts">
	import { onMount } from 'svelte';
	import { listInstances } from '$lib/helpers/instances';
	import type { Account } from '$lib/types';
	import Button from '$lib/components/ui/button/button.svelte';
	import Card from '$lib/components/ui/card/card.svelte';
	import CardContent from '$lib/components/ui/card/card-content.svelte';
	import CardHeader from '$lib/components/ui/card/card-header.svelte';
	import CardTitle from '$lib/components/ui/card/card-title.svelte';
	import Avatar from '$lib/components/ui/avatar/avatar.svelte';

	let { account, status, isLoggingIn, onLogin, onLogout, onLaunch } = $props();

	let instances = $state<string[]>([]);

	onMount(async () => {
		await loadInstances();
	});

	async function loadInstances(): Promise<void> {
		instances = await listInstances();
	}
</script>

<div class="flex justify-between items-center mb-6">
	<div class="flex-1">
		<p>Status: <b>{status}</b></p>
	</div>

	{#if account}
		<div class="flex items-center gap-3">
			<Avatar
				src={`https://mc-heads.net/avatar/${account.name}`}
				alt={`Avatar for ${account.name}`}
			/>
			<div class="flex flex-col items-end">
				<span class="font-medium">{account.name}</span>
				<Button onclick={onLogout} variant="outline" size="sm">Log Out</Button>
			</div>
		</div>
	{:else}
		<Button onclick={onLogin} disabled={isLoggingIn}>
			{isLoggingIn ? 'Authenticating...' : 'Login with Microsoft'}
		</Button>
	{/if}
</div>

<h1 class="text-3xl font-bold mb-6">Instances</h1>

{#if instances.length > 0}
	<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
		{#each instances as name}
			<Card>
				<CardHeader>
					<CardTitle>{name}</CardTitle>
				</CardHeader>
				<CardContent>
					<Button
						onclick={() => onLaunch(name)}
						disabled={isLoggingIn}
						class="w-full"
						variant="default"
					>
						{account ? 'Play' : 'Login & Play'}
					</Button>
				</CardContent>
			</Card>
		{/each}
	</div>
{:else}
	<Card>
		<CardContent class="p-8 text-center">
			<p class="text-muted-foreground">No instances found. Go to Install page to create one!</p>
		</CardContent>
	</Card>
{/if}
