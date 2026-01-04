<script lang="ts">
	import { cn } from '$lib/utils';
	import type { HTMLInputAttributes } from 'svelte/elements';

	interface CheckboxProps extends HTMLInputAttributes {
		class?: string;
	}

	let { class: className = '', ...restProps }: CheckboxProps = $props();
</script>

<input
	type="checkbox"
	class={cn('h-4 w-4 rounded border-input bg-background text-primary focus:ring-2 focus:ring-ring focus:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50', className)}
	{...restProps}
/>
