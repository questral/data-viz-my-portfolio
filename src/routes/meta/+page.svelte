
<script>
    import * as d3 from "d3";
    import { onMount } from "svelte";

    let data = $state([]);

    $effect(() => {
            const loadData = async () => {
                // data = await d3.csv("loc.csv");
                data = await d3.csv("loc.csv", row => ({
                    ...row,
                    line: Number(row.line), // or just +row.line
                    depth: Number(row.depth),
                    length: Number(row.length),
                    date: new Date(row.date + "T00:00" + row.timezone),
                    datetime: new Date(row.datetime)
                }));
                console.log(data);
            };

            loadData();
        });
    
    

    
</script>

<svelte:head>
	<title>meta</title>
</svelte:head>

<h1>meta</h1>
<p>Total lines of code: {data.length}</p>