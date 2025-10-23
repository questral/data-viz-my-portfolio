<script lang="ts">
	import AQIChart from '$lib/AQIChart.svelte';
	import AQIChart2 from '$lib/AQIChart2.svelte';
	import * as d3 from 'd3';
	import {onMount} from 'svelte';

	const datasets = {
		avalon:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BAvalon%5D_daily-avg.csv',
		glassport_high_street:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BGlassport%20High%20Street%5D_daily-avg.csv',
		lawrenceville:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BLawrenceville%5D_daily-avg.csv',
		liberty_sahs:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BLiberty%20(SAHS)%5D_daily-avg.csv',
		manchester:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BManchester%5D_daily-avg.csv',
		north_braddock:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BNorth%20Braddock%5D_daily-avg.csv',
		parkway_east_near_road:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BParkway%20East%20(Near%20Road)%5D_daily-avg.csv',
		usa_pennsylvania_pittsburgh:
			'https://dig.cmu.edu/datavis-fall-2025/assignments/data/%5BUSA-Pennsylvania-Pittsburgh%5D_daily-avg.csv'
	};

	const datasetNames = {
		avalon: 'Avalon',
		glassport_high_street: 'High Street, Glassport',
		lawrenceville: 'Lawrenceville',
		liberty_sahs: 'South Allegheny HS, Liberty',
		manchester: 'Manchester',
		north_braddock: 'North Braddock',
		parkway_east_near_road: 'Parkway East (near road)',
		usa_pennsylvania_pittsburgh: 'Pittsburgh'
	}

	let keys = Object.keys(datasets);
	
	// async function getLengths(datasets: object) {
	// 	for (let k of keys) {
	// 		let size = await d3.csv(datasets[k]).then((d) => d.length);
	// 		console.log(size);
	// 	}
	// }

	// getLengths(datasets);

	let selectedDataset: keyof typeof datasets = $state('lawrenceville');

	let showScatter = $state(false);

	const data = $derived.by(() =>
		d3.csv(datasets[selectedDataset], (d: any) => ({
			city: d.City,
			country: d.Country,
			mainPollutant: d['Main pollutant'],
			pm25: +d['PM2.5'],
			state: d.State,
			stationName: d['Station name'],
			timestamp: new Date(d['Timestamp(UTC)']),
			usAqi: +d['US AQI']
		}))
	);
	
	let textColors = ["#000000","#ffffff"];
	let colors = [
		{"name":"Good","min":0,"max":50,"color":"#9cd84e","textColor":textColors[0]},
		{"name":"Moderate","min":51,"max":100,"color":"#facf39","textColor":textColors[0]},
		{"name":"Unhealthy for Sensitive Groups","min":101,"max":150,"color":"#f99049","textColor":textColors[0]},
		{"name":"Unhealthy","min":151,"max":200,"color":"#f65e5f","textColor":textColors[1]},
		{"name":"Very Unhealthy","min":201,"max":300,"color":"#a070b6","textColor":textColors[1]},
		{"name":"Hazardous","min":301,"color":"#a06a7b","textColor":textColors[1]}
	]

</script>

<svelte:head>
	<title>a3 part 2</title>
</svelte:head>

{#await data}
	<!-- promise is pending -->
	<p>loading data...</p>
{:then data}
	<!-- promise was fulfilled or not a Promise -->
	<h1>AQI Chart 2</h1>
	<p><em>Try brushing across the scatterplot!</em></p>
	<AQIChart2 data={data} showScatter={showScatter} />
	<p><b>Number of records:</b> {data.length}</p>
{:catch error}
	<!-- promise was rejected -->
	<p>Something went wrong: {error.message}</p>
{/await}

<select bind:value={selectedDataset}>
	{#each keys as k}
		<option value={k}>{datasetNames[k]}</option>
	{/each}
</select>

<input type="checkbox" id="scatterShower" name="scatterShower" bind:checked={showScatter}>
<label for="scatterShower">Show raw data</label>

<div class="legend">
	<section class="usAqiTitle">US AQI:</section>
	<section class="swatches">
	{#each colors as c}
			<div class="swatch" style="background:{c.color}; color: {c.textColor}">
				<div class="name">{c.name}</div>
				<div class="range">{c.min}{c.max ? "-"+c.max : "+"}</div>
			</div>
			
	{/each}
	</section>
</div>


<style>
	* {
		font-family: sans-serif;
	}

	.legend {
		width: 100%;
		display: flex;
		flex-direction: row;
		flex-wrap: nowrap;

		margin-top: 20px;
		margin-bottom: 50px;
	}

	section.usAqiTitle {
		/* background: red; */
		text-align: center;
		width: 60px;
		/* min-width: 60px; */
		font-size: 0.8em;
		align-items: center;
	}

	section.swatches {
		display: inline-flex;
		width: 93%;
		/* max-width: 1000px; */
		flex-wrap: wrap;
		flex-direction: row;
		/* background: red; */
		/* justify-content: space-between; */
		gap: 5px;
	}

	.swatch {
		width: 130px;
		height: 50px;
		border-radius: 10px;
		display: flex;
		flex-direction: row;
		font-size: 0.7em;
		align-items: center;
		box-sizing: border-box;
		padding: 10px;
		justify-content: space-between;
	}

	.name {
		justify-content: center;
		text-align: center;
		width: auto;
	}

	.range {
		justify-content: center;
		text-align: right;
		width: 100%;
		/* background: red; */
	}

</style>
