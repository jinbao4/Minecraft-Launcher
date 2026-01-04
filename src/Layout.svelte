<script lang="ts">
	import { navItems } from "./lib/nav";
	import { onMount } from "svelte";

	export let children: any;

	let currentLocation: string;

	onMount(() => {
		currentLocation = window.location.pathname;
	});

	function navigate(location: string) {
		window.history.pushState({}, "", location);
		currentLocation = location;
	}

	onMount(() => {
		const handlePopState = () => {
			currentLocation = window.location.pathname;
		};
		window.addEventListener("popstate", handlePopState);
		return () => {
			window.removeEventListener("popstate", handlePopState);
		};
	});
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
			<button
				onclick={() => { navigate(item['location']); }}
				class:active={currentLocation === item['location']}
			>
				{item['label']}
			</button>
		{/each}
	</nav>
</div>

<div class="flex min-h-screen min-w-[320px] items-center justify-center">
	<div class="w-full max-w-[1280px] px-8 text-center">
		<div>
			{@render children()}
		</div>
	</div>
</div>

<style>
	nav button {
		padding: 0.5rem 1rem;
		border-radius: 0.375rem;
		background: transparent;
		border: none;
		cursor: pointer;
		transition: background-color 0.2s;
		color: #666;
	}

	nav button:hover {
		background-color: #f3f4f6;
	}

	nav button.active {
		background-color: #e5e7eb;
		color: #111;
		font-weight: 500;
	}
</style>
