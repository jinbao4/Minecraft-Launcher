<script lang="ts">
	import Button from "$lib/components/ui/button/button.svelte";
import { navItems } from "./lib/nav";

	let { children, currentLocation, navigate } = $props();
</script>

<svelte:head>
	<style>
		body {
			margin: 0;
		}
	</style>
</svelte:head>

<div class="absolute left-1/2 top-0 transform -translate-x-1/2 mt-4">
	<nav class="mb-8 flex justify-center gap-2">
		{#each navItems as item, i (i)}
			<Button
				onclick={() => { navigate(item['location']); }}
				class={currentLocation === item['location'] ? "active" : ""}
			>
				{item['label']}
			</Button>
		{/each}
	</nav>
</div>

<div class="flex min-h-screen min-w-[320px] items-center justify-center dark">
	<div class="w-full max-w-[1280px] px-8 text-center">
		<div>
			{@render children()}
		</div>
	</div>
</div>