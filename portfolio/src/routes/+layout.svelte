<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<svelte:head><link rel="icon" href={favicon} /><title>yuans-dev</title></svelte:head>
<div class={`flex h-screen w-screen flex-col items-center justify-start gap-4 bg-slate-950 `}>
	{@render children()}
</div>
