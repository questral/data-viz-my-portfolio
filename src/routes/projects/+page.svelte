<!-- <nav>
    <ul>
        <li><a href="../index.html">home</a></li>
        <li><a href="#">projects</a></li>
        <li><a href="../contacts/index.html">contacts</a></li>
        <li><a href="../resume/index.html">resume</a></li>
        <li><a href="https://github.com/questral" target="_blank">my git</a></li>
    </ul>
</nav> -->

<script>
    import projects from '$lib/projects.json';
    import Project from "$lib/Project.svelte";
    import Pie from '$lib/Pie.svelte';
    import * as d3 from 'd3';

    // let pieData = [
    //     { value: 1, label: "apples" },
    //     { value: 2, label: "oranges" },
    //     { value: 3, label: "mangos" },
    //     { value: 4, label: "pears" },
    //     { value: 5, label: "limes" },
    //     { value: 5, label: "cherries" }
    // ];

    let query = $state("");
    // const filteredProjects = $derived(
    //     projects.filter(project => {
    //         if (query) {
    //             return project.title.includes(query.toLowerCase());
    //         }

    //         return true;
    //     })
    // );
    const filteredProjects = $derived(
        projects.filter(project => {
            let values = Object.values(project).join("\n").toLowerCase();
            return values.includes(query.toLowerCase());
        })
    );

    // let rolledData = d3.rollups(filteredProjects, v => v.length, d => d.year);
    // let pieData = rolledData.map(([year, count]) => {
    //     return { value: count, label: year };
    // });
    
    const pieData = $derived.by(() => {
        let rolledData = d3.rollups(filteredProjects, v => v.length, d => d.year);
        return rolledData.map(([year, count]) => {
            return { value: count, label: year };
        });
    });

    // let rolledData = d3.rollups(filteredProjects, v => v.length, d => d.year);
	// let pieData = rolledData.map(([year, count]) => {
	// 	return { value: count, label: year };
	// });

    let selectedYearIndex = $state(-1);
    const selectedYear = $derived(
        selectedYearIndex > -1 ? pieData[selectedYearIndex].label : null
    );
    const filteredByYear = $derived(
        filteredProjects.filter(project => {
            if (selectedYear) {
                return project.year === selectedYear;
            }
            return true;
        })
    );

</script>
<!-- <pre>{ JSON.stringify(projects, null, "\t") }</pre> -->

<svelte:head>
	<title>projects</title>
</svelte:head>


<!-- <Pie /> -->
<Pie data={pieData} bind:selectedIndex={selectedYearIndex} />
<!-- {selectedYear} -->

<h1>{projects.length} projects</h1>

<input type="search" bind:value={query}
       aria-label="Search projects" placeholder="🔍 Search projects…" />

<div class="projects">
    <!-- {#each projects as p} -->
    {#each filteredByYear as p}
        <Project data={p} />
        <!-- <article> -->
            <!-- <h2>Lorem ipsum dolor sit.</h2> -->
            <!-- <h2>{p.title}</h2> -->
            <!-- <img src="https://dig.cmu.edu/datavis-fall-2024/labs/2/images/empty.svg" alt=""> -->
            <!-- <img src={p.image} alt=""> -->
            <!-- <p>Lorem ipsum dolor sit amet consectetur adipisicing elit. Magnam dolor quos, quod assumenda explicabo odio, nobis ipsa laudantium quas eum veritatis ullam sint porro minima modi molestias doloribus cumque odit.</p> -->
            <!-- <p>{p.description}</p> -->
        <!-- </article> -->
    {/each}
    <!-- <article>
        <h2>Nulla quasi aliquid delectus.</h2>
        <img src="https:/dig.cmu.edu/datavis-fall-2024/labs/2/images/empty.svg" alt="">
        <p>Rem beatae eum accusamus. Quos eius fugit, quaerat aliquid tempore excepturi autem blanditiis neque, numquam consequuntur adipisci, asperiores facilis tenetur molestias dolor totam odit aut facere modi maxime. At, nostrum?</p>
    </article>
    <article>
        <h2>Corporis reprehenderit tempore dicta!</h2>
        <img src="https:/dig.cmu.edu/datavis-fall-2024/labs/2/images/empty.svg" alt="">
        <p>Consequatur deleniti, a possimus, magni necessitatibus dolorem ipsa veniam cupiditate dignissimos culpa cum, impedit ullam! Nulla iure omnis voluptatum, soluta odit at, numquam hic saepe vitae, iusto quasi! Aliquam, voluptatibus.</p>
    </article>
    <article>
        <h2>Tenetur magnam ratione dolor?</h2>
        <img src="https:/dig.cmu.edu/datavis-fall-2024/labs/2/images/empty.svg" alt="">
        <p>Repellendus neque velit nemo earum. Saepe explicabo voluptatibus distinctio doloribus amet suscipit rem a, fugiat nisi omnis quas tempore. Totam a laboriosam, veniam rerum culpa animi. Dicta quae consectetur magnam.</p>
    </article>
    <article>
        <h2>Quas neque dolor eaque.</h2>
        <img src="https:/dig.cmu.edu/datavis-fall-2024/labs/2/images/empty.svg" alt="">
        <p>Commodi nam quae quam laudantium, illo fugiat voluptate, nisi saepe libero deleniti enim minus? Labore illum quod sapiente delectus natus dolorem iure fugit saepe. Culpa nobis perspiciatis id atque dolores.</p>
    </article>
    <article>
        <h2>Cumque ipsa eum enim.</h2>
        <img src="https:/dig.cmu.edu/datavis-fall-2024/labs/2/images/empty.svg" alt="">
        <p>Laboriosam possimus iusto vitae, commodi doloremque labore fuga ea aut magni vero nobis dignissimos veniam eos ipsum sequi, ducimus pariatur odit aperiam nisi explicabo qui, atque molestiae dicta. Ab, modi?</p>
    </article>
    <article>
        <h2>Hic vitae atque eaque.</h2>
        <img src="https:/dig.cmu.edu/datavis-fall-2024/labs/2/images/empty.svg" alt="">
        <p>Provident quia dignissimos non omnis? Quisquam minima suscipit dignissimos at molestias eligendi aliquid ullam vitae aperiam quidem, itaque unde consequuntur dolorum autem similique enim consectetur qui? Blanditiis, amet! Possimus, reiciendis!</p>
    </article>
    <article>
        <h2>Vero tempora dicta id.</h2>
        <img src="https:/dig.cmu.edu/datavis-fall-2024/labs/2/images/empty.svg" alt="">
        <p>Ea obcaecati laudantium quasi soluta voluptate perferendis mollitia facilis nemo, accusamus esse itaque necessitatibus sint quidem ipsa neque expedita natus reprehenderit eius, harum voluptatibus dolores culpa. Sunt maxime eveniet alias.</p>
    </article>
    <article>
        <h2>Consequuntur magni illo provident!</h2>
        <img src="https:/dig.cmu.edu/datavis-fall-2024/labs/2/images/empty.svg" alt="">
        <p>Soluta cupiditate nostrum ad aliquam exercitationem labore error, ratione qui maiores enim consectetur earum corporis quibusdam recusandae, quos non natus officiis quia dicta voluptates voluptatum. Consectetur minima iste quidem laborum!</p>
    </article>
    <article>
        <h2>Facilis nam ducimus natus.</h2>
        <img src="https:/dig.cmu.edu/datavis-fall-2024/labs/2/images/empty.svg" alt="">
        <p>Eligendi dignissimos officia, non perspiciatis quaerat similique? Odit asperiores aperiam sapiente voluptates provident itaque tenetur, alias fugit a mollitia aliquam non atque eaque iste distinctio blanditiis ipsa libero. Aut, molestiae?</p>
    </article>
    <article>
        <h2>Aliquid voluptatem corrupti blanditiis?</h2>
        <img src="https:/dig.cmu.edu/datavis-fall-2024/labs/2/images/empty.svg" alt="">
        <p>Neque suscipit laboriosam quod iusto doloribus, nostrum quia natus fugit, sequi eos voluptate odit voluptates nihil facere alias eaque. Explicabo quo nihil, eligendi officiis inventore commodi! Modi eaque inventore a.</p>
    </article>
    <article>
        <h2>Sequi minima nihil quasi!</h2>
        <img src="https:/dig.cmu.edu/datavis-fall-2024/labs/2/images/empty.svg" alt="">
        <p>Quam eum fugiat ea expedita iste minima aliquam corporis quibusdam nemo dignissimos quae, sequi sunt ipsum, obcaecati beatae nulla iusto facilis at quisquam repellendus voluptate voluptas ipsam. Inventore, consequuntur odit!</p>
    </article> -->
</div>