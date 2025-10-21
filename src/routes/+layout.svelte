<script>
	import favicon from '$lib/assets/favicon.svg';
	import { page } from '$app/state';

	let pages = [
		{url: "./", title: "home"},
		{url: "./projects", title: "projects"},
		{url: "./contacts", title: "contacts"},
		{url: "./resume", title: "resume"},
		{url: "./meta", title: "meta"},
		{url: "https://github.com/questral", title: "my git"},
	];

	let { children } = $props();

	let colorScheme = $state("light dark");
	let root = globalThis?.document?.documentElement;
	$effect(() => root?.style.setProperty("color-scheme", colorScheme));

	let localStorage = globalThis.localStorage ?? {};
	if (localStorage.colorScheme) {
		colorScheme = localStorage.colorScheme;
	}
	$effect(() => {
		localStorage.colorScheme = colorScheme;
	});


</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>


<nav>
	{#each pages as p }
		<!-- Link here, using {p.url} for the URL and {p.title} for the title -->
		<a href={ p.url } class:current={ "." + page.route.id === p.url } 
		   target={ p.url.startsWith("http") ? "_blank" : null }>{ p.title }</a>
	{/each}
</nav>

<!-- {colorScheme} -->
<label class="color-scheme">
	Theme:
	<select bind:value={ colorScheme }>
		<option value="light dark">Automatic</option> <!-- (${matchMedia("(prefers-color-scheme: dark)").matches ? 'Dark' : 'Light'})</option> -->
		<option value="light">Light</option>
		<option value="dark">Dark</option>
	</select>
</label>

{@render children?.()}

<!-- YOLO <slot /> -->
<!-- { JSON.stringify(page) } -->




<style>
	label.color-scheme {
		position: absolute;
		top: 1rem;
		right: 1rem;

		font-family: inherit;
		font-size: 80%;
	}
</style>
