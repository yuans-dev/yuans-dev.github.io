<script>
	import Prism from '$lib/components/Prism.svelte';
	import favicon from '$lib/assets/favicon.svg';
</script>

<div class="relative flex h-screen min-h-200 w-full items-center justify-start">
	<div class="pointer-events-none absolute z-0 -translate-x-8 px-50 py-24 opacity-10">
		<img src={favicon} alt="Favicon" class="h-48 w-48" />
	</div>
	<div class="absolute z-1 h-full w-full"><Prism></Prism></div>
	<div class="pointer-events-none absolute z-2 px-50 py-24">
		<img src={favicon} alt="Favicon" class="h-48 w-48" />
	</div>
</div>
