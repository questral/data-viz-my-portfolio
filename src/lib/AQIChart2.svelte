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

	
	// set the dimensions and margins of the graph
	const margin = {top: 10, right: 10, bottom: 30, left: 30},
		width = 850 - margin.left - margin.right,
		height = 400 - margin.top - margin.bottom,
		w1 = (width*0.7) - 0,
		w2 = (width*0.3) - 0;
		// svgX2 = w1 + margin.left;

	// append the svg object to the body of the page
	const svg = $derived(d3.select("#my_dataviz")
		.append("svg")
			.attr("width", w1 + margin.left + margin.right) // + margin.right
			.attr("height", height + margin.top + margin.bottom)
		.append("g")
			.attr("transform", `translate(${margin.left}, ${margin.top})`));
	
	const svg2 = $derived(d3.select("#my_dataviz2")
		.append("svg")
			.attr("width", w2 + margin.left + margin.right) // + margin.right
			.attr("height", height + margin.top + margin.bottom)
		.append("g")
			.attr("transform", `translate(${margin.left}, ${margin.top})`)
			);

	// function sortDictList(dictList, k) {
	// 	dictList.sort((a, b) => a.k - b.k);
	// }

	let selectedDots = $state([]);

	let hoverIndex = $state(-1);

	let colors = [
		{"name":"Good","min":0,"max":50,"color":"#9cd84e"},
		{"name":"Moderate","min":51,"max":100,"color":"#facf39"},
		{"name":"Unhealthy for Sensitive Groups","min":101,"max":150,"color":"#f99049"},
		{"name":"Unhealthy","min":151,"max":200,"color":"#f65e5f"},
		{"name":"Very Unhealthy","min":201,"max":300,"color":"#a070b6"},
		{"name":"Hazardous","min":301,"color":"#a06a7b"}
	]

	let pollutantColors = {
		"PM2.5": "#000000",
		"Ozone": "#ff0000",
	}

	//Read the data
	function loadData2() {
		// console.log($state.snapshot(selectedDots));
		svg2.selectAll("circle").remove();
		svg2.selectAll("rect").remove();
		svg2.selectAll("path").remove();
		svg2.selectAll(".axis").remove();

		let minAqi = +d3.min(data, (d) => d.usAqi);
		let maxAqi = +d3.max(data, (d) => d.usAqi) + 5;

		let maxNDots = 25;
		let trimmedSelectedDots = selectedDots.slice(0,maxNDots);

		let minRank = 0;
		let maxRank = trimmedSelectedDots.length+1;


		const x2 = $derived(d3.scaleLinear()
			.domain([minRank, maxRank])
			.range([0,w2]));
		svg2.append("g")
			.attr("class", "axis")
			.attr("transform", `translate(0, ${height})`)
			.call(d3.axisBottom(x2)
				.ticks(maxRank <= 15 ? maxRank : maxRank/2));
				
		const y2 = d3.scaleLinear()
			.domain([minAqi, maxAqi])
			.range([height,0]);
		svg2.append("g")
			.attr("class", "axis")
			.call(d3.axisLeft(y2)
				.ticks(8));
		
		
		// const xGridLines = $derived(svg2.selectAll("xGrid")
		// 	.data(x2.ticks(maxRank))
		// 	.join("line")
		// 	.attr("x1", d => x2(d))
		// 	.attr("x2", d => x2(d))
		// 	.attr("y1", 0)
		// 	.attr("y2", height)
		// 	.attr("stroke", "#e0e0e0")
		// 	.attr("stroke-width", 1)
		// );
		
		svg2.selectAll("yGrid")
			.data(y2.ticks(maxAqi/20).slice(1))
			.join("line")
			.attr("x1", 0)
			.attr("x2", w2)
			.attr("y1", d => y2(d))
			.attr("y2", d => y2(d))
			.attr("stroke", "#e0e0e0")
			.attr("stroke-width", 1)


		const band = d3.scaleLinear()
			.domain([0,maxAqi])
			.range([height,0])

		svg2.append("g")
			.selectAll("rect.colorful")
			.data(colors)
			.join("rect")
				.attr("class", "colorful")
				.style("fill", (c) => c.color)
				.attr("stroke", "none")
				.attr("x", 0)
				.attr("y", (c) => {
					let maxVal = (c.max ? c.max : maxAqi);
					return Math.max(band(maxVal), 0);
				})
				.attr("width", w2)
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
		

		const tooltip = d3.select("#my_dataviz2")
			.append("div")
			.style("opacity", 0)
			.attr("class", "tooltip")
			.style("background-color", "white")
			.style("border", "solid")
			.style("border-width", "1px")
			.style("padding", "10px")
			.style("position", "absolute")

		const mouseover = function(event, d) {
			tooltip
			.style("opacity", 1);

			hoverIndex = trimmedSelectedDots.indexOf(d);

			dot
				.style("opacity", (d) => (hoverIndex == trimmedSelectedDots.indexOf(d) ? 1 : 0.3));
			hitbox
				.style("opacity", (d) => (hoverIndex == trimmedSelectedDots.indexOf(d) ? 0.2 : 0));
			// console.log(hoverIndex);
			// console.log('mouseover');
		}

		const mousemove = function(event, d) {
			let dayStr = (d.timestamp.getDay()+1) < 10 ? '0'+(d.timestamp.getDay()+1) : (d.timestamp.getDay()+1);
			let monthStr = (d.timestamp.getMonth()+1) < 10 ? '0'+(d.timestamp.getMonth()+1) : (d.timestamp.getMonth()+1);
			let tooltipStr = `<em>${d.timestamp.getFullYear()}&#8211;${monthStr}&#8211;${dayStr}</em> <br>Main pollutant: <b>${d.mainPollutant}</b>`;
			tooltip
			.html(tooltipStr)
			.style("left", (event.x)+10 + "px")
			.style("top", (event.y)-60 + "px");

			// dot
			// 	.style("opacity", (d) => (hoverIndex == trimmedSelectedDots.indexOf(d) ? 1 : 0.3));
		}

		const mouseleave = function(event,d) {
			tooltip
			// .transition()
			// .duration(200)
			.html("")
			.style("left", 0 + "px")
			.style("top", 0 + "px")
			.style("opacity", 0);

			dot
				.style("opacity", 1); //(d) => (hoverIndex == trimmedSelectedDots.indexOf(d) ? 1 : 0.3));
			hitbox
				.style("opacity", 0); //(d) => (hoverIndex == trimmedSelectedDots.indexOf(d) ? 1 : 0));

			hoverIndex = -1;
		}

		
		// Add dots
		const dot = svg2.append('g')
			.selectAll("dot")
			.data(trimmedSelectedDots)
			.join("circle")
				.attr("cx", (d) => x2(d.index+1) )
				.attr("cy", (d) => y2(d.usAqi) )
				.attr("r", 2)
				.style("fill", (d) => pollutantColors[d.mainPollutant])
				.style("opacity", 1)
		
		//hitbox
		const hitbox = svg2.append('g')
			.selectAll("dot")
			.data(trimmedSelectedDots)
			.join("circle")
				.attr("cx", (d) => x2(d.index+1) )
				.attr("cy", (d) => y2(d.usAqi) )
				.attr("r", 12)
				.style("fill", (d) => pollutantColors[d.mainPollutant])
				.style("stroke", "solid")
				.style("stroke-width", "1px")
				.style("opacity", 0)
			.on("mouseover", mouseover )
			.on("mousemove", mousemove )
			.on("mouseleave", mouseleave );

	}

	function loadData() {
		// https://raw.githubusercontent.com/holtzy/data_to_viz/master/Example_dataset/2_TwoNum.csv

		// console.log(data);

		svg.selectAll("circle").remove();
		svg.selectAll("rect").remove();
		svg.selectAll("path").remove();

		let minDate = +d3.min(data, (d) => d.timestamp);
		let maxDate = +d3.max(data, (d) => d.timestamp);
		// console.log(minDate, maxDate);
		let minAqi = +d3.min(data, (d) => d.usAqi);
		let maxAqi = +d3.max(data, (d) => d.usAqi) + 5;
		// console.log(minAqi, maxAqi);

		// Add X axis
		const x = d3.scaleUtc()
			.domain([minDate, maxDate])
			.range([0,w1]);
		svg.append("g")
			.attr("transform", `translate(0, ${height})`)
			.call(d3.axisBottom(x));

		// Add Y axis
		const y = d3.scaleLinear()
			.domain([minAqi, maxAqi])
			.range([height,0]);
		svg.append("g")
			.call(d3.axisLeft(y)
			.ticks(8));
		
		
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
		

		svg.selectAll("yGrid")
			.data(y.ticks(maxAqi/20).slice(1))
			.join("line")
			.attr("x1", 0)
			.attr("x2", w1)
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
				.attr("y", (c) => {
					let maxVal = (c.max ? c.max : maxAqi);
					return Math.max(band(maxVal), 0);
				})
				.attr("width", w1)
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
			const dot = svg.append('g')
				.selectAll("dot")
				.data(data)
				.join("circle")
					.attr("cx", (d) => x(d.timestamp) )
					.attr("cy", (d) => y(d.usAqi) )
					.attr("r", 1)
					.style("fill", (d) => pollutantColors[d.mainPollutant])
					.style("opacity", 1)
			

			// Create the brush behavior.
			svg.call(d3.brushX()
				.extent([[margin.left, 0], [w1, height]])
				.on("start brush end", ({selection}) => {
				// let selectedDots = [];
				if (selection) {
					// console.log(selection);
					const [[x0, y0], [x1, y1]] = [[selection[0],0], [selection[1],height]];
					// const [[x0, y0], [x1, y1]] = selection;

					selectedDots = dot
						.style("opacity", 0.3)
						.filter(d => x0 <= x(d.timestamp) && x(d.timestamp) < x1
								&& y0 <= y(d.usAqi) && y(d.usAqi) < y1)
						.style("opacity", 1)
						.data();
					selectedDots.sort((a,b) => b.usAqi - a.usAqi);
					selectedDots.map((d) => {d["index"] = selectedDots.indexOf(d)});
					loadData2();
					
					
				} else {
					dot
						.style("fill", (d) => pollutantColors[d.mainPollutant])
						.style("opacity", 1);
					selectedDots = [];
					loadData2();
				}

				// Inform downstream cells that the selection has changed.
				svg.property("value", selectedDots).dispatch("input");
			}));



		}
		// console.log(sortedSelectedDots);
	}

	$effect(() => {
		loadData();
	})

	onMount(() => {
		loadData();
		loadData2();
	})

</script>

<pre>
<!-- {JSON.stringify(data[0], null, 2)} -->
</pre>


<section class="graphs">
	<div id="my_dataviz"></div>
	<div id="my_dataviz2"></div>
</section>
<!-- <div class="tooltip">Lorem ipsum dolor sit amet consectetur adipisicing elit. Necessitatibus</div> -->

<style>
	.graphs {
		display: flex;
		flex-direction: row;
		flex-wrap: nowrap;
		gap: 0;
	}

	svg {
		font-family: sans-serif;
		overflow: visible;
	}

	/* .tooltip {
		max-width: 200px;
		background-color: white;
		border: solid;
		border-width: 1px;
		border-radius: 5px;
		padding: 10px;
	} */
</style>
