<!-- Reusable Navbar for CP06 Profile Editor -->

<script lang="ts">
	import {
		Navbar,
		NavBrand,
		NavUl,
		NavLi,
		DarkMode
	} from 'flowbite-svelte';

	import { page } from '$app/stores';
	import { derived } from 'svelte/store';

	const activeUrl = derived(page, ($page) => $page.url.pathname);
	let activeUrlValue: string;
	$: activeUrl.subscribe(value => activeUrlValue = value);
</script>

<Navbar>
	<NavBrand href="/">
		<span
			class="text-4xl pl-10 font-semibold whitespace-nowrap text-purple-500 dark:text-purple-400"
			>CP06 <span class="text-black dark:text-white">Profile Editor</span></span
		>
	</NavBrand>

	<NavUl activeUrl={activeUrlValue}>
		<NavLi href="/">Profile Editor</NavLi>
	</NavUl>

	<!-- ✅ Valid placement of toggle outside <NavUl> to avoid nesting inside <button> -->
	<div class="flex items-center pr-4">
		<DarkMode />
	</div>
</Navbar>