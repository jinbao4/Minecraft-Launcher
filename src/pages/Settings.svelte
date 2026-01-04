<script lang="ts">
	import Card from '$lib/components/ui/card/card.svelte';
	import CardContent from '$lib/components/ui/card/card-content.svelte';
	import CardHeader from '$lib/components/ui/card/card-header.svelte';
	import CardTitle from '$lib/components/ui/card/card-title.svelte';
</script>

<div class="settings-content">
	<h1 class="settings-title">Settings</h1>

	<Card>
		<CardHeader>
			<CardTitle>General</CardTitle>
		</CardHeader>
		<CardContent>
			<p>Settings coming soon...</p>
		</CardContent>
	</Card>
</div>

<style>
	.settings-content {
		flex: 1;
		display: flex;
		flex-direction: column;
		padding: 24px;
		gap: 24px;
	}

	.settings-title {
		font-size: 28px;
		font-weight: 600;
		color: #1a1a1a;
	}
</style>
