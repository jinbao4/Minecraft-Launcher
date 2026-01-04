<script lang="ts">
	import { onMount } from 'svelte';
	import { installInstance } from '$lib/helpers/instances';
	import { setupInstallListeners } from '$lib/helpers/events';
	import { fetchVersionManifest, filterVersions, type VersionEntry } from '$lib/helpers/versions';
	import Select from '$lib/components/ui/select/select.svelte';
	import Button from '$lib/components/ui/button/button.svelte';
	import Card from '$lib/components/ui/card/card.svelte';
	import CardContent from '$lib/components/ui/card/card-content.svelte';
	import CardHeader from '$lib/components/ui/card/card-header.svelte';
	import CardTitle from '$lib/components/ui/card/card-title.svelte';
	import Checkbox from '$lib/components/ui/checkbox/checkbox.svelte';

	let { account, status, isLoggingIn, onLogin, onLogout } = $props();

	let allVersions = $state<VersionEntry[]>([]);
	let filteredVersions = $derived.by(
		() => filterVersions(allVersions, searchQuery, includeSnapshots)
	);
	let selectedVersion = $state('');
	let instanceName = $state('');
	let installStatus = $state('');
	let isLoading = $state(true);
	let isInstalling = $state(false);
	let includeSnapshots = $state(false);
	let searchQuery = $state('');

	onMount(async () => {
		await fetchVersions();

		setupInstallListeners(
			(statusMessage) => {
				installStatus = statusMessage;
				if (statusMessage === 'Installation Complete!') {
					isInstalling = false;
				}
			},
			(error) => {
				installStatus = `Error: ${error}`;
				isInstalling = false;
			}
		);
	});

	async function fetchVersions() {
		try {
			const manifest = await fetchVersionManifest();
			allVersions = manifest.versions;
			selectedVersion = manifest.latest.release;
			instanceName = `Minecraft ${manifest.latest.release}`;
			isLoading = false;
		} catch (error) {
			installStatus = `Failed to fetch versions: ${error}`;
			isLoading = false;
		}
	}

	async function handleInstall() {
		if (!instanceName.trim()) {
			installStatus = 'Please enter an instance name';
			return;
		}
		if (!selectedVersion) {
			installStatus = 'Please select a version';
			return;
		}

		isInstalling = true;
		installStatus = `Starting installation of ${instanceName} (${selectedVersion})...`;

		try {
			await installInstance(instanceName, selectedVersion, selectedVersion);
		} catch (error) {
			installStatus = `Installation failed: ${error}`;
			isInstalling = false;
		}
	}

	function handleVersionChange() {
		if (selectedVersion && !instanceName) {
			instanceName = `Minecraft ${selectedVersion}`;
		}
	}
</script>

<div class="flex justify-between items-center mb-4">
	<div class="flex-1">
		<p>Status: <b>{status || installStatus}</b></p>
	</div>

	{#if account}
		<div class="flex items-center gap-2">
			<img
				src="https://mc-heads.net/avatar/{account.name}"
				alt="Avatar for {account.name}"
				width="32"
				height="32"
				class="rounded-full"
			/>
			<div class="flex flex-col items-end">
				<span>{account.name}</span>
				<Button onclick={onLogout} variant="outline" size="sm">Log Out</Button>
			</div>
		</div>
	{:else}
		<Button onclick={onLogin} disabled={isLoggingIn}>
			{isLoggingIn ? 'Authenticating...' : 'Login with Microsoft'}
		</Button>
	{/if}
</div>

<h1 class="text-3xl font-bold mb-6">Install Minecraft</h1>

{#if isLoading}
	<Card>
		<CardContent class="p-4">
			<p>Loading versions...</p>
		</CardContent>
	</Card>
{:else}
	<div class="flex gap-4">
		<div class="flex-1 space-y-4">
			<Card>
				<CardHeader>
					<CardTitle>Installations</CardTitle>
				</CardHeader>
				<CardContent class="space-y-4">
					<div class="space-y-2">
						<label for="instance-name" class="block text-sm font-medium">Instance Name</label>
						<input
							id="instance-name"
							type="text"
							bind:value={instanceName}
							placeholder="Enter instance name"
							class="flex h-10 w-full rounded-md border border-input bg-background px-3 py-2 text-sm"
							disabled={isInstalling}
						/>
					</div>

					<div class="flex items-center gap-2">
						<Checkbox
							checked={includeSnapshots}
							id="snapshots"
							onchange={(e) => {
								const target = e.target as HTMLInputElement | null;
								includeSnapshots = target?.checked ?? false;
							}}
						/>
						<label for="snapshots" class="text-sm font-medium">Include Snapshots</label>
					</div>

					<div class="space-y-2">
						<label for="search" class="block text-sm font-medium">Search Versions</label>
						<input
							id="search"
							type="text"
							bind:value={searchQuery}
							placeholder="Search versions..."
							class="flex h-10 w-full rounded-md border border-input bg-background px-3 py-2 text-sm"
							disabled={isInstalling}
						/>
					</div>

					<div class="space-y-2">
						<label for="version-select" class="block text-sm font-medium">Minecraft Version</label>
						<Select
							value={selectedVersion}
							onchange={handleVersionChange}
							disabled={isInstalling}
						>
							{#each filteredVersions as version}
								<option value={version.id}>{version.id}</option>
							{/each}
						</Select>
					</div>

					<Button
						onclick={handleInstall}
						disabled={isInstalling || !instanceName.trim() || !selectedVersion}
						class="w-full"
					>
						{isInstalling ? 'Installing...' : 'Install'}
					</Button>

					{#if installStatus}
						<div class="p-3 rounded-md bg-muted text-sm">
							{installStatus}
						</div>
					{/if}
				</CardContent>
			</Card>
		</div>

		<div class="flex-1">
			<Card>
				<CardHeader>
					<CardTitle>Available Versions</CardTitle>
				</CardHeader>
				<CardContent>
					<div class="max-h-96 overflow-y-auto space-y-2">
						{#each filteredVersions as version (version.id)}
							<button
								class="w-full flex items-center justify-between p-2 rounded-md border border-border hover:bg-accent transition-colors disabled:opacity-50 disabled:cursor-not-allowed"
								onclick={() => (selectedVersion = version.id)}
								disabled={isInstalling}
								class:active={selectedVersion === version.id}
							>
								<div class="flex flex-col items-start">
									<span class="font-medium">{version.id}</span>
									<span class="text-xs text-muted-foreground">{version.type}</span>
								</div>
								{#if selectedVersion === version.id}
									<span class="text-sm text-primary">✓</span>
								{/if}
							</button>
						{/each}

						{#if filteredVersions.length === 0}
							<p class="text-center text-muted-foreground text-sm p-4">
								No versions found matching your search.
							</p>
						{/if}
					</div>
				</CardContent>
			</Card>
		</div>
	</div>
{/if}

<style>
	.active {
		background-color: hsl(var(--primary));
		color: hsl(var(--primary-foreground));
		border-color: hsl(var(--primary));
	}
</style>
