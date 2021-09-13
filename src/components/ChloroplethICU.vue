<template>
	<div>
		<!-- <ul v-for="county in counties" v-bind:key="county.fips">
			<li>{{ county.county }}</li>
			<p>{{ county.state }}</p>
		</ul> -->
		<!-- <div id="icu-map"></div> -->
		<!-- <div class="viz"></div> -->
		<div id="content">
	<h1>U.S. Daily Cigarette Smoking Rate, 1996-2012</h1>
	<h2 class="year"></h2>
	<div class="slider"></div>
	<div id="map"></div>
	<p>Source: Institute for Health Metrics and Evaluation.</p>
</div>
	</div>
</template>

<script>
import * as d3 from "d3";
import * as queue from "d3-queue";
import * as topojson from "topojson-client";
import { nest } from "d3-collection";
export default {
	name: "ChloroplethICU",
	props: {},
	data: function() {
		return {
			apiBaseUrl: "https://api.covidactnow.org/v2/",
			apiKey: "55783d149c7b461b87d3df4de4ffa9a1",
			covidData: [],
			mapData: [],
		};
	},
	computed: {
		covidCountyDataUrl() {
			return this.apiBaseUrl + "counties.json?apiKey=" + this.apiKey;
		},
	},
	async created() {
		// this.getData();
		// this.getMapData();

		await this.getData();
		await this.getMapData();

		// this.drawMap();
		this.drawMap2();
		// this.drawMap3();
	},
	mounted() {
		// this.drawMap();
		// this.drawMap2();
	},
	methods: {
		async getData() {
			try {
				let response = await fetch(this.covidCountyDataUrl);
				this.covidData = await response.json();
			} catch (error) {
				console.log(error);
			}
		},
		async getMapData() {
			try {
				let response = await fetch("us.json");
				this.mapData = await response.json();
			} catch (error) {
				console.log(error);
			}
		},
		drawMap3() {
			var margin = {
					top: 10,
					bottom: 10,
					left: 10,
					right: 10,
				},
				width1 = parseInt(d3.select(".viz").style("width")),
				width = width1 - margin.left - margin.right,
				mapRatio = 0.5,
				height = width * mapRatio,
				active = d3.select(null);

			var svg = d3
				.select(".viz")
				.append("svg")
				.attr("class", "center-container")
				.attr("height", height + margin.top + margin.bottom)
				.attr("width", width + margin.left + margin.right);

			svg.append("rect")
				.attr("class", "background center-container")
				.attr("height", height + margin.top + margin.bottom)
				.attr("width", width + margin.left + margin.right)
				.on("click", clicked);

			// Promise.resolve(d3.json("us.json")).then(ready);

			var projection = d3
				.geoAlbersUsa()
				.translate([width / 2, height / 2])
				.scale(width);

			var path = d3.geoPath().projection(projection);

			var g = svg
				.append("g")
				.attr("class", "center-container center-items us-state")
				.attr(
					"transform",
					"translate(" + margin.left + "," + margin.top + ")"
				)
				.attr("width", width + margin.left + margin.right)
				.attr("height", height + margin.top + margin.bottom);
			ready(this.mapData);
			function ready(us) {
				console.log(us);
				g.append("g")
					.attr("id", "counties")
					.selectAll("path")
					.data(topojson.feature(us, us.objects.counties).features)
					.enter()
					.append("path")
					.attr("d", path)
					.attr("class", "county-boundary")
					.on("click", reset);

				g.append("g")
					.attr("id", "states")
					.selectAll("path")
					.data(topojson.feature(us, us.objects.states).features)
					.enter()
					.append("path")
					.attr("d", path)
					.attr("class", "state")
					.on("click", clicked);

				g.append("path")
					.datum(
						topojson.mesh(us, us.objects.states, function(a, b) {
							return a !== b;
						})
					)
					.attr("id", "state-borders")
					.attr("d", path);
			}

			function clicked(d) {
				if (d3.select(".background").node() === this) return reset();

				if (active.node() === this) return reset();

				active.classed("active", false);
				active = d3.select(this).classed("active", true);

				var bounds = path.bounds(d),
					dx = bounds[1][0] - bounds[0][0],
					dy = bounds[1][1] - bounds[0][1],
					x = (bounds[0][0] + bounds[1][0]) / 2,
					y = (bounds[0][1] + bounds[1][1]) / 2,
					scale = 0.9 / Math.max(dx / width, dy / height),
					translate = [width / 2 - scale * x, height / 2 - scale * y];

				g.transition()
					.duration(750)
					.style("stroke-width", 1.5 / scale + "px")
					.attr(
						"transform",
						"translate(" + translate + ")scale(" + scale + ")"
					);
			}

			function reset() {
				active.classed("active", false);
				active = d3.select(null);

				g.transition()
					.delay(100)
					.duration(750)
					.style("stroke-width", "1.5px")
					.attr(
						"transform",
						"translate(" + margin.left + "," + margin.top + ")"
					);
			}
		},
		// drawMap() {
		// 	console.log("drawmap");
		// 	// console.log(this.covidData[0]);
		// 	let width = 960,
		// 		height = 600;
		// 	let color_domain = [
		// 		500,
		// 		1000,
		// 		1500,
		// 		2000,
		// 		2500,
		// 		3000,
		// 		3500,
		// 		4000,
		// 		4500,
		// 		5000,
		// 		5500,
		// 		6000,
		// 	];
		// 	let ext_color_domain = [
		// 		0,
		// 		500,
		// 		1000,
		// 		1500,
		// 		2000,
		// 		2500,
		// 		3000,
		// 		3500,
		// 		4000,
		// 		4500,
		// 		5000,
		// 		5500,
		// 		6000,
		// 	];
		// 	let legend_labels = [
		// 		"< 500",
		// 		"500+",
		// 		"1000+",
		// 		"1500+",
		// 		"2000+",
		// 		"2500+",
		// 		"3000+",
		// 		"3500+",
		// 		"4000+",
		// 		"4500+",
		// 		"5000+",
		// 		"5500+",
		// 		"6000+",
		// 	];
		// 	let color = d3
		// 		.scaleThreshold()
		// 		.domain(color_domain)
		// 		.range([
		// 			"#dcdcdc",
		// 			"#d0d6cd",
		// 			"#bdc9be",
		// 			"#aabdaf",
		// 			"#97b0a0",
		// 			"#84a491",
		// 			"#719782",
		// 			"#5e8b73",
		// 			"#4b7e64",
		// 			"#387255",
		// 			"#256546",
		// 			"#125937",
		// 			"#004d28",
		// 		]);

		// 	let div = d3
		// 		.select("#icu-map")
		// 		.append("div")
		// 		.attr("class", "tooltip")
		// 		.style("opacity", 0);

		// 	let svg = d3
		// 		.select("#icu-map")
		// 		.append("svg")
		// 		.attr("width", width)
		// 		.attr("height", height)
		// 		.style("margin", "-15px auto");
		// 	let path = d3.geoPath();

		// 	// queue()
		// 	// 	.defer(d3.json, "us.json")
		// 	// 	.defer(d3.csv, "data.csv")
		// 	// 	.await(ready);
		// 	// ready(this.mapData, this.covidData);
		// 	// function ready(mapData, covidData) {
		// 	// 	console.log(mapData.objects);
		// 	var pairIcuBedsWithId = {};
		// 	var pairNameWithId = {};

		// 	//Moves selction to front
		// 	d3.selection.prototype.moveToFront = function() {
		// 		return this.each(function() {
		// 			this.parentNode.appendChild(this);
		// 		});
		// 	};

		// 	//Moves selction to back
		// 	d3.selection.prototype.moveToBack = function() {
		// 		return this.each(function() {
		// 			var firstChild = this.parentNode.firstChild;
		// 			if (firstChild) {
		// 				this.parentNode.insertBefore(this, firstChild);
		// 			}
		// 		});
		// 	};

		// 	this.covidData.forEach(function(d) {
		// 		pairIcuBedsWithId[d.fips] = +d.actuals.icuBeds.capacity;
		// 		pairNameWithId[d.fips] = d.county;
		// 		// console.log(d.actuals.icuBeds.capacity);
		// 	});

		// 	// console.log(pairNameWithId);
		// 	let that = this;

		// 	// console.log(this.mapData);

		// 	svg.append("g")
		// 		.attr("class", "county")
		// 		.selectAll("path")
		// 		.data(
		// 			topojson.feature(
		// 				this.mapData,
		// 				this.mapData.objects.counties
		// 			).features
		// 		)
		// 		.enter()
		// 		.append("path")
		// 		.attr("d", path)
		// 		.style("fill", function(d) {
		// 			return color(pairIcuBedsWithId[d.id]);
		// 		})
		// 		.style("opacity", 0.8)
		// 		.on("mouseover", function(d) {
		// 			var sel = d3.select(this);
		// 			sel.moveToFront();
		// 			d3.select(this)
		// 				.transition()
		// 				.duration(300)
		// 				.style({
		// 					opacity: 1,
		// 					stroke: "black",
		// 					"stroke-width": 1.5,
		// 				});
		// 			div.transition()
		// 				.duration(300)
		// 				.style("opacity", 1);
		// 			div.text(
		// 				pairNameWithId[d.id] + ": " + pairIcuBedsWithId[d.id]
		// 			)
		// 				.style("left", d3.event.pageX + "px")
		// 				.style("top", d3.event.pageY - 30 + "px");
		// 		})
		// 		.on("mouseout", function() {
		// 			var sel = d3.select(this);
		// 			sel.moveToBack();
		// 			d3.select(this)
		// 				.transition()
		// 				.duration(300)
		// 				.style({
		// 					opacity: 0.8,
		// 					stroke: "white",
		// 					"stroke-width": 1,
		// 				});
		// 			div.transition()
		// 				.duration(300)
		// 				.style("opacity", 0);
		// 		});
		// 	// }

		// 	var legend = svg
		// 		.selectAll("g.legend")
		// 		.data(ext_color_domain)
		// 		.enter()
		// 		.append("g")
		// 		.attr("class", "legend");

		// 	var ls_w = 73,
		// 		ls_h = 20;

		// 	legend
		// 		.append("rect")
		// 		.attr("x", function(d, i) {
		// 			return width - i * ls_w - ls_w;
		// 		})
		// 		.attr("y", 550)
		// 		.attr("width", ls_w)
		// 		.attr("height", ls_h)
		// 		.style("fill", function(d, i) {
		// 			return color(d);
		// 		})
		// 		.style("opacity", 0.8);

		// 	legend
		// 		.append("text")
		// 		.attr("x", function(d, i) {
		// 			return width - i * ls_w - ls_w;
		// 		})
		// 		.attr("y", 590)
		// 		.text(function(d, i) {
		// 			return legend_labels[i];
		// 		});

		// 	var legend_title = "Number of independent farms";

		// 	svg.append("text")
		// 		.attr("x", 10)
		// 		.attr("y", 540)
		// 		.attr("class", "legend_title")
		// 		.text(function() {
		// 			return legend_title;
		// 		});
		// },
		drawMap2() {
			var margin = { top: 20, right: 20, bottom: 20, left: 20 };
			var width = 800 - margin.left - margin.right,
				height = 500 - margin.top - margin.bottom,
				formatPercent = d3.format(".1%");

			var svg = d3
				.select("#map")
				.append("svg")
				.attr("width", width + margin.left + margin.right)
				.attr("height", height + margin.top + margin.bottom)
				.append("g")
				.attr(
					"transform",
					"translate(" + margin.left + "," + margin.top + ")"
				);

			var tooltip = d3
				.select("body")
				.append("div")
				.attr("class", "tooltip")
				.style("opacity", 0);

			// queue()
			// 	.defer(d3.csv, "smoking-data.csv")
			// 	.defer(d3.json, "us.json")
			// 	.await(ready);

			var legendText = ["", "10%", "", "15%", "", "20%", "", "25%"];
			var legendColors = [
				"#fff7bc",
				"#fee391",
				"#fec44f",
				"#fe9929",
				"#ec7014",
				"#cc4c02",
				"#993404",
				"#662506",
			];

			ready('', this.covidData, this.mapData);
			function ready(error, data, us) {
				var counties = topojson.feature(us, us.objects.counties);
				// console.log(data);

				data.forEach(function(d) {
					// d.year = +d.year;
					d.fips = +d.fips;
					// d.rate = +d.rate;
					d.county = +d.county;
					d.openIcuBeds = +d.actuals.icuBeds.capacity;
				});
				// console.log(data[0]);

				// var dataByCountyByYear = d3
				// 	.nest()
				// 	.key(function(d) {
				// 		return d.fips;
				// 	})
				// 	.key(function(d) {
				// 		return d.year;
				// 	})
				// 	.map(data);

				// counties.features.forEach(function(county) {
				// 	county.properties.years = dataByCountyByYear[+county.id];
				// });

				var color = d3.scaleThreshold()
					.domain([10, 12.5, 15, 17.5, 20, 22.5, 25])
					.range([
						"#fff7bc",
						"#fee391",
						"#fec44f",
						"#fe9929",
						"#ec7014",
						"#cc4c02",
						"#993404",
						"#662506",
					]);

				var projection = d3.geoAlbersUsa()
					.translate([width / 2, height / 2]);

				var path = d3.geoPath().projection(projection);

				var countyShapes = svg
					.selectAll(".county")
					.data(counties)
					.enter()
					.append("path")
					.attr("class", "county")
					.attr("d", path);

				countyShapes
					.on("mouseover", function(d) {
						tooltip
							.transition()
							.duration(250)
							.style("opacity", 1);
						tooltip
							.html(
								"<p><strong>" +
									d.properties.years[1996][0].county +
									", " +
									d.properties.years[1996][0].state +
									"</strong></p>" +
									"<table><tbody><tr><td class='wide'>Smoking rate in 1996:</td><td>" +
									formatPercent(
										d.properties.years[1996][0].rate / 100
									) +
									"</td></tr>" +
									"<tr><td>Smoking rate in 2012:</td><td>" +
									formatPercent(
										d.properties.years[2012][0].rate / 100
									) +
									"</td></tr>" +
									"<tr><td>Change:</td><td>" +
									formatPercent(
										(d.properties.years[2012][0].rate -
											d.properties.years[1996][0].rate) /
											100
									) +
									"</td></tr></tbody></table>"
							)
							.style("left", d3.event.pageX + 15 + "px")
							.style("top", d3.event.pageY - 28 + "px");
					})
					.on("mouseout", function(d) {
						tooltip
							.transition()
							.duration(250)
							.style("opacity", 0);
					});

				svg.append("path")
					.datum(
						topojson.feature(us, us.objects.states, function(a, b) {
							return a !== b;
						})
					)
					.attr("class", "states")
					.attr("d", path);

				var legend = svg.append("g").attr("id", "legend");

				var legenditem = legend
					.selectAll(".legenditem")
					.data(d3.range(8))
					.enter()
					.append("g")
					.attr("class", "legenditem")
					.attr("transform", function(d, i) {
						return "translate(" + i * 31 + ",0)";
					});

				legenditem
					.append("rect")
					.attr("x", width - 240)
					.attr("y", -7)
					.attr("width", 30)
					.attr("height", 6)
					.attr("class", "rect")
					.style("fill", function(d, i) {
						return legendColors[i];
					});

				legenditem
					.append("text")
					.attr("x", width - 240)
					.attr("y", -10)
					.style("text-anchor", "middle")
					.text(function(d, i) {
						return legendText[i];
					});

				function update(year) {
					slider.property("value", year);
					d3.select(".year").text(year);
					countyShapes.style("fill", function(d) {
						return color(d.properties.years[year][0].rate);
					});
				}

				var slider = d3
					.select(".slider")
					.append("input")
					.attr("type", "range")
					.attr("min", 1996)
					.attr("max", 2012)
					.attr("step", 1)
					.on("input", function() {
						var year = this.value;
						update(year);
					});

				update(1996);
			}

			d3.select(self.frameElement).style("height", "685px");
		},
	},
};
</script>

<style lang="scss">
// path {
// 	stroke: white;
// 	stroke-width: 1px;
// }

// body {
// 	font-family: "Proxima Nova", sans-serif;
// }

// .county {
// 	font: 14px sans-serif;
// 	font-weight: bold;
// 	fill:orange;
// }

// .legend {
// 	font-size: 14px;
// 	font-family: "Proxima Nova", sans-serif;
// }

// .legend_title {
// 	font-size: 14px;
// 	font-family: "Proxima Nova", sans-serif;
// 	font-weight: bold;
// }

// div.tooltip {
// 	position: absolute;
// 	left: 75px;
// 	text-align: center;
// 	height: 16px;
// 	padding: 10px;
// 	font-size: 14px;
// 	background: #ffffff;
// 	border: 1px solid #989898;
// 	pointer-events: none;
// }

// p {
// 	font-family: "Proxima Nova", sans-serif;
// 	font-size: 10px;
// 	margin: 20px 0 0 10px;
// }

// @media (max-width: 400px) {
// 	.d3map {
// 		display: none;
// 	}
// }
// ===========
body,
h1,
h2,
h3,
p {
	margin: 0;
	padding: 0;
	font-family: "Source Sans Pro", sans-serif;
	font-size: 1em;
	color: #333;
	font-weight: 400;
}

#content {
	margin: 5px;
	padding: 20px;
	width: 805px;
	border: 1px solid #ccc;
}

#map {
	margin: 10px 0px 0px 0px;
	padding: 0px;
}

h1,
h2 {
	line-height: 1em;
	font-size: 1.75em;
	font-weight: 900;
	color: #000;
}

h2.year {
	margin: 10px 0px 0px 0px;
	font-size: 1.3em;
	font-weight: 700;
}

p {
	margin: 10px 0px 0px 0px;
}

.county {
	fill: #666;
	stroke: #fff;
	stroke-linejoin: round;
	width:100px;
	height:100px;
}
.states:hover {
	// fill:orange;
}

.states {
	fill: none;
	fill:#999;
	stroke: #fff;
	stroke-linejoin: round;
}

input {
	display: block;
	width: 200px;
	margin: 10px 0px 0px 0px;
}

#legend text {
	font-size: 0.9em;
	color: #333;
	font-weight: 400;
}

.tooltip {
	position: absolute;
	padding: 7px;
	font-size: 0.9em;
	pointer-events: none;
	background: #fff;
	border: 1px solid #ccc;
	border-radius: 4px;

	-moz-box-shadow: 3px 3px 10px 0px rgba(0, 0, 0, 0.25);
	-webkit-box-shadow: 3px 3px 10px 0px rgba(0, 0, 0, 0.25);
	box-shadow: 3px 3px 10px 0px rgba(0, 0, 0, 0.25);
}

.tooltip p {
	margin: 0;
	padding: 0;
}

.tooltip table {
	margin: 0;
	padding: 0;
	border-collapse: collapse;
}

.wide {
	width: 140px;
}

// ============
// .background {
// 	fill: none;
// 	pointer-events: all;
// }

// #states {
// 	fill: #aaa;
// }

// #states .active {
// 	// display: none;
// }

// #state-borders {
// 	fill: none;
// 	stroke: #fff;
// 	stroke-width: 1.5px;
// 	stroke-linejoin: round;
// 	stroke-linecap: round;
// 	pointer-events: none;
// }

// .county-boundary {
// 	fill: #aaa;
// 	stroke: #fff;
// 	stroke-width: 0.5px;
// }

// .county-boundary:hover,
// .state:hover {
// 	fill: orange;
// }
</style>
