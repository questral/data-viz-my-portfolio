<script lang="ts">
	import * as d3 from 'd3';
	import {onMount} from 'svelte';

	interface Item {
		city: string;
		country: string;
		mainPollutant: string;
		pm25: number;
		state: string;
		stationName: string;
		timestamp: Date;
		usAqi: number;
	}

	
	// properties this component accepts
	const { data, showScatter }: { data: Item[], showScatter: boolean } = $props();

	// just for debugging; can be removed
	// $inspect(data);
	// console.log(data);

	// // set the dimensions and margins of the graph
	// const margin = {top: 10, right: 30, bottom: 30, left: 60},
	// 	width = 1000 - margin.left - margin.right,
	// 	height = 400 - margin.top - margin.bottom;
	
	// const xScale = $derived(d3.scaleUtc()
	// 	.domain([+d3.min(data, (d) => d.timestamp.getFullYear()),+d3.max(data, (d) => d.timestamp.getFullYear())])
	// 	.range([0,width]));
	
	// const yScale = $derived(d3.scaleLinear()
	// 	.domain([+d3.min(data, (d) => d.usAqi), +d3.max(data, (d) => d.usAqi)])
	// 	.range([height,0]));
	
	// const line = $derived(d3
	// 	.line()
	// 	.x((d) => xScale(new Date(d.timestamp)))
	// 	.y((d) => yScale(d.usAqi))
	// );

	// const linePath = $derived(line(data));

	// // Create axis generators
	// const xAxis = $derived(d3.axisBottom(xScale).ticks(width / 80).tickSizeOuter(10));
  	// const yAxis = $derived(d3.axisLeft(yScale).ticks(height / 40));

	// // Refs for axis elements
	// let xAxisRef = $state();
  	// let yAxisRef = $state();

	// 	// Update axes when scales change
	// $effect(() => {
	// 	if (xAxisRef) {
	// 	d3.select(xAxisRef).call(xAxis);
	// 	}
	// });
	
	// $effect(() => {
	// 	if (yAxisRef) {
	// 	d3.select(yAxisRef).call(yAxis);
	// 	}
	// });

	// set the dimensions and margins of the graph
	const margin = {top: 10, right: 30, bottom: 30, left: 60},
		width = 850 - margin.left - margin.right,
		height = 400 - margin.top - margin.bottom;

	// append the svg object to the body of the page
	const svg = $derived(d3.select("#my_dataviz")
		.append("svg")
			.attr("width", width + margin.left + margin.right)
			.attr("height", height + margin.top + margin.bottom)
		.append("g")
			.attr("transform", `translate(${margin.left}, ${margin.top})`));

	
	//Read the data
	function loadData() {
		// https://raw.githubusercontent.com/holtzy/data_to_viz/master/Example_dataset/2_TwoNum.csv

		// console.log(data);

		svg.selectAll("circle").remove();
		svg.selectAll("rect").remove();
		svg.selectAll("path").remove();

		// Add X axis
		// const x = d3.scaleLinear()
		// .domain([0, 4000])
		// .range([ 0, width ]);
		let minDate = +d3.min(data, (d) => d.timestamp);
		let maxDate = +d3.max(data, (d) => d.timestamp);
		// console.log(minDate, maxDate);
		const x = d3.scaleUtc()
			.domain([minDate, maxDate])
			.range([0,width]);
		svg.append("g")
			.attr("transform", `translate(0, ${height})`)
			.call(d3.axisBottom(x));

		// Add Y axis
		// const y = d3.scaleLinear()
		// .domain([0, 500000])
		// .range([height, 0]);
		let minAqi = +d3.min(data, (d) => d.usAqi);
		let maxAqi = +d3.max(data, (d) => d.usAqi) + 5;
		// console.log(minAqi, maxAqi);
		const y = d3.scaleLinear()
			.domain([minAqi, maxAqi])
			.range([height,0]);
		svg.append("g")
			.call(d3.axisLeft(y)
			.ticks(8));

		// let monthGroups = d3.groups(data, (d) => d.timestamp.getMonth());
		// console.log(monthGroups);
		// let avgs = monthGroups.map(([month, lines]) => {
		// 	let avgAqi = d3.mean(lines, line => line.usAqi);
		// 	let ret = {
		// 		month: month,
		// 		avgAqi: avgAqi,
		// 		timestamp: lines[0].timestamp
		// 	}
		// 	return ret;
		// });
		// console.log(avgs);
		let yearGroups = d3.groups(data, (d) => d.timestamp.getFullYear(), (d) => d.timestamp.getMonth());
		// console.log(yearGroups);
		let avgs = [];
		for (let year of yearGroups) {
			// console.log('year', year);
			let monthAvgs = year[1].map(([month, lines]) => {
				let avgAqi = d3.mean(lines, line => line.usAqi);
				let allAqis = lines.map(line => line.usAqi);
				// console.log('allAqis', allAqis);
				let percentile10 = d3.quantile(allAqis, 0.1);
				let percentile90 = d3.quantile(allAqis, 0.9);
				let ret = {
					month: month,
					avgAqi: avgAqi,
					timestamp: lines[0].timestamp,
					percentile10: percentile10,
					percentile90: percentile90
				}
				avgs.push(ret);
				return ret;
			})
		}
		// console.log(avgs);

		let colors = [
			{"name":"Good","min":0,"max":50,"color":"#9cd84e"},
			{"name":"Moderate","min":51,"max":100,"color":"#facf39"},
			{"name":"Unhealthy for Sensitive Groups","min":101,"max":150,"color":"#f99049"},
			{"name":"Unhealthy","min":151,"max":200,"color":"#f65e5f"},
			{"name":"Very Unhealthy","min":201,"max":300,"color":"#a070b6"},
			{"name":"Hazardous","min":301,"color":"#a06a7b"}
		]

		svg.selectAll("yGrid")
			.data(y.ticks(maxAqi/20).slice(1))
			.join("line")
			.attr("x1", 0)
			.attr("x2", width)
			.attr("y1", d => y(d))
			.attr("y2", d => y(d))
			.attr("stroke", "#e0e0e0")
			.attr("stroke-width", 1)
		
		// const band = d3.scaleBand([0, 51, 101, 151, 201, 301])
		// 	.domain(["Good", "Moderate", "Unhealthy for Sensitive Groups", "Unhealthy", "Very Unhealthy", "Hazardous"]);
		const band = d3.scaleLinear()
			.domain([0,maxAqi])
			.range([height,0])

		svg.append("g")
			.selectAll("rect.colorful")
			.data(colors)
			.join("rect")
				.attr("class", "colorful")
				.style("fill", (c) => c.color)
				.attr("stroke", "none")
				.attr("x", 0)
				// .attr("y", (c) => y(c.max ? c.max : height))
				// .attr("y", (c) => Math.max(band((c.max ? c.max : maxAqi)), 0))
				.attr("y", (c) => {
					let maxVal = (c.max ? c.max : maxAqi);
					return Math.max(band(maxVal), 0);
				})
				.attr("width", width)
				// .attr("height", (c) => Math.max(band(c.min - 1) - band(c.max ? c.max : maxAqi), 0))
				.attr("height", (c) => {
					let maxVal = (c.max ? c.max : 350);
					let bandY = band(maxVal);
					let amountOver = Math.max(0 - bandY, 0);
					// console.log(c.name, bandY, amountOver);
					let baseHeight = -(band(maxVal) - band(Math.max(c.min-1, 0)));
					let newHeight = Math.max(baseHeight - amountOver, 0);
					// console.log(c.name, baseHeight);
					return newHeight;
				} )
				.attr("opacity", 0.5)
	
		//add confidence interval
		svg.append("path")
			.datum(avgs)
			.attr("fill", "#000000")
			.attr("opacity", 0.15)
			.attr("stroke", "none")
			.attr("d", d3.area((d) => x(d.timestamp), (d) => y(d.percentile10), (d) => y(d.percentile90))
				// .x((d) => x(d.timestamp))
				// .y0(function(d) { return y(d.CI_right) })
				// .y1(function(d) { return y(d.CI_left) })
			)
		
		//add line
		svg.append("path")
			.datum(avgs)
			.attr("fill", "none")
			.attr("stroke", "black")
			.attr("stroke-width", 1.5)
			.attr("d", d3.line((d) => x(d.timestamp), (d) => y(d.avgAqi))
				// .y([0,1])
				// .x(function(d) { console.log(d); return x(d.timestamp);} )
				// .y(function(d) { return y(d.usAqi);} )
				// .x(function(d) { return x(d.date) })
				// .y(function(d) { return y(d.value) })
			)
		
		if (showScatter) {
			// Add dots
			svg.append('g')
				.selectAll("dot")
				.data(data)
				.join("circle")
					.attr("cx", (d) => x(d.timestamp) )
					.attr("cy", (d) => y(d.usAqi) )
					.attr("r", 1)
					.style("fill", "#000000");
		}
		
	}

	$effect(() => {
		loadData();
	})

	onMount(() => {
		loadData();
	})

</script>

<pre>
<!-- {JSON.stringify(data[0], null, 2)} -->
</pre>

<!-- <svg id='plot'
	{width}
	{height}
	viewBox="0 0 {width} {height}"
	style:max-width="100%"
	style:height="auto">

	<g bind:this={yAxisRef} transform="translate({margin.left},0)"></g>
	<g bind:this={xAxisRef} transform="translate(0,{height - margin.bottom})"></g>

	<g transform="translate({margin.left},0)">
		{#each yScale.ticks() as tick}
		  {#if tick !== 0}
			<line
			  stroke="currentColor"
			  stroke-opacity="0.1"
			  x1={0}
			  x2={width - margin.left - margin.right}
			  y1={yScale(tick)}
			  y2={yScale(tick)}
			/>
		  {/if}
		{/each}
	</g>

  	<path fill="none" stroke="steelblue" stroke-width="1.5" d={linePath} />
</svg> -->

<div id="my_dataviz"></div>

<style>
	svg {
		font-family: sans-serif;
		overflow: visible;
	}
</style>
