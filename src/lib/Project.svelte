<script>
    let { data = {}, hLevel = 2 } = $props();
    // let { hLevel = 2 } = $props();
</script>

<article>
    <svelte:element this={"h" + hLevel}>{ data.title }</svelte:element>

    <!-- <h2>{data.title}</h2> -->
    <img src={data.image} alt="">
    <div>
        <p>{data.description}</p>
        <p>{data.year}</p>
    </div>
</article>