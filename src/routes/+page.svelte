<!-- <nav>
    <ul>
        <li><a href="#">home</a></li>
        <li><a href="projects/index.html">projects</a></li>
        <li><a href="contacts/index.html">contacts</a></li>
        <li><a href="resume/index.html">resume</a></li>
        <li><a href="https://github.com/questral" target="_blank">my git</a></li>
        <li><a href="a1_data_viz/A1_report.html" target="_blank">Assignment 1</a></li>
    </ul>
</nav> -->

<script>
    import projects from '$lib/projects.json';
    import Project from "$lib/Project.svelte";
    // let { hLevel = 3 } = $props();
    let globalData = $state();
</script>

<h1>my name</h1>

<p><em>is kai and im a student practicing working on learning knowing how to work on reading and understanding the art of categorizing and displaying interesting sets of data :)</em> Lorem ipsum dolor sit amet consectetur, adipisicing elit. Ut provident eius natus est, magni dolorum corporis aspernatur maxime iusto voluptas fugiat praesentium harum, cumque, quis suscipit doloribus magnam ipsam! Voluptas.</p>

<img src="/images/scrappy doo bisque.webp" alt="deranged evil lobster">


{#await fetch("https://api.github.com/users/questral") }
	<p>Loading...</p>
{:then response}
	{#await response.json()}
		<p>Decoding...</p>
	{:then data}
		<!-- <p>The data is { JSON.stringify(data) }</p> -->
        <section>
            <h2>my stats</h2>
            <dl>
                <dt>public repos:</dt>
                <dd>{ data.public_repos }</dd>
            </dl>
        </section>

	{:catch error}
		<p class="error">
			Something went wrong: {error.message}
		</p>
	{/await}
{:catch error}
	<p class="error">
		Something went wrong: {error.message}
	</p>
{/await}




<h2>latest projs</h2>
<div class="projects">
    {#each projects.slice(0, 3) as p}
        <Project data={p} hLevel=3/>
    {/each}
</div>