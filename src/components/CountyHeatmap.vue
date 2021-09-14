<template>
	<div class="heatmap">
		<div v-if="!covidData.length">
			<Spinner />
		</div>
		<div v-else>
			<div id="map"></div>
		</div>
	</div>
</template>

<script>
import Spinner from "@/components/Spinner.vue";
import * as d3 from "d3";
// import * as d3 from "d3@3.0.0";
// import * as queue from "d3-queue";
import * as topojson from "topojson-client";
// import { nest } from "d3-collection";
export default {
	components: {
		Spinner,
	},
	data: function() {
		return {
			apiBaseUrl: "https://api.covidactnow.org/v2/",
			apiKey: "55783d149c7b461b87d3df4de4ffa9a1",
			covidData: [],
			mapData: [],
			pairBedsWithId: {},
			pairNameWithId: {},
		};
	},
	computed: {
		covidCountyDataUrl() {
			return this.apiBaseUrl + "counties.json?apiKey=" + this.apiKey;
		},
	},
	async created() {
		await this.getData();
		await this.getMapData();
		await this.populateSortedData();
		await this.drawMap();
	},
	mounted() {},
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
				let response = await fetch("us2.json");
				this.mapData = await response.json();
			} catch (error) {
				console.log(error);
			}
		},
		populateSortedData() {
			let that = this;
			this.covidData.forEach(function(d) {
				let fipsStringToNumber = parseInt(d.fips);
				that.pairBedsWithId[fipsStringToNumber] = +(
					d.actuals.icuBeds.capacity -
					d.actuals.icuBeds.currentUsageTotal
				);
				that.pairNameWithId[fipsStringToNumber] = d.county;
				// console.log(d.actuals.icuBeds.capacity);
			});
		},
		drawMap() {
			let that = this;
			// queue()
			// 	.defer(d3.json, "us.json")
			// 	.defer(
			// 		d3.json,
			// 		"https://api.covidactnow.org/v2/counties.json?apiKey=55783d149c7b461b87d3df4de4ffa9a1"
			// 	)
			// 	.await(ready);

			// ready('', this.mapData, this.covidData);
			// function ready(error, us, data) {
			var width = 960,
				height = 600;
			var color_domain = [
				5,
				10,
				15,
				20,
				25,
				30,
				35,
				10000
			];
			var ext_color_domain = [
				0,
				5,
				10,
				15,
				20,
				25,
				30,
				35,
			];
			var legend_labels = [
				"0",
				"5+",
				"10+",
				"15+",
				"20+",
				"25+",
				"30+",
				"35+",
			];
			var color = d3.scale
				.threshold()

				.domain(color_domain)
				.range([
					"#662506",
					"#993404", 
					"#cc4c02",
					"#ec7014", 
					"#fe9929", 
					"#fec44f",
					"#fee391", 
					"#fff7bc",   
				]);

			var div = d3
				.select("#map")
				.append("div")
				.attr("class", "tooltip")
				.style("opacity", 0);

			var svg = d3
				.select("#map")
				.append("svg")
				.attr("width", width)
				.attr("height", height)
				.style("margin", "-15px auto");
			// var projection = d3.geo
			// 	.albersUsa()
			// 	.translate([width / 2, height / 2]);
			// var path = d3.geo.path().projection(projection);
			var path = d3.geo.path();
			// console.log(data);
			// var pairBedsWithId = {};
			// var pairNameWithId = {};

			//Moves selction to front
			d3.selection.prototype.moveToFront = function() {
				return this.each(function() {
					this.parentNode.appendChild(this);
				});
			};

			//Moves selction to back
			d3.selection.prototype.moveToBack = function() {
				return this.each(function() {
					var firstChild = this.parentNode.firstChild;
					if (firstChild) {
						this.parentNode.insertBefore(this, firstChild);
					}
				});
			};

			// this.CovidData.forEach(function(d) {
			// 	// pairRateWithId[d.id] = +d.rate;
			// 	pairBedsWithId[d.fips] = +(
			// 		d.actuals.icuBeds.capacity -
			// 		d.actuals.icuBeds.currentUsageTotal
			// 	);
			// 	pairNameWithId[d.fips] = d.county;
			// 	// console.log(d.actuals.icuBeds.capacity);
			// });
			// console.log(pairBedsWithId);
			// console.log(that.pairNameWithId);
			svg.append("g")
				.attr("class", "county")
				.selectAll("path")
				.data(
					topojson.feature(
						that.mapData,
						that.mapData.objects.counties
					).features
				)
				.enter()
				.append("path")
				.attr("d", path)
				.style("fill", function(d) {
					return color(that.pairBedsWithId[d.id]);
				})
				.style("opacity", 0.8)
				.on("mouseover", function(d) {
					var sel = d3.select(this);
					sel.moveToFront();
					d3.select(this)
						.transition()
						.duration(300)
						.style({
							opacity: 1,
							stroke: "black",
							"stroke-width": 1.5,
						});
					div.transition()
						.duration(300)
						.style("opacity", 1);
					div.text(
						that.pairNameWithId[d.id] +
							": " +
							that.pairBedsWithId[d.id]
					)
						.style("left", d3.event.pageX + "px")
						.style("top", d3.event.pageY - 30 + "px");
				})
				.on("mouseout", function() {
					var sel = d3.select(this);
					sel.moveToBack();
					d3.select(this)
						.transition()
						.duration(300)
						.style({
							opacity: 0.8,
							stroke: "white",
							"stroke-width": 0,
						});
					div.transition()
						.duration(300)
						.style("opacity", 0);
				});
			// }
			svg.append("path")
				.datum(
					topojson.feature(
						that.mapData,
						that.mapData.objects.states,
						function(a, b) {
							return a !== b;
						}
					)
				)
				.attr("class", "states")
				.attr("d", path);

			var legend = svg
				.selectAll("g.legend")
				.data(ext_color_domain)
				.enter()
				.append("g")
				.attr("class", "legend");

			var ls_w = 73,
				ls_h = 20;

			legend
				.append("rect")
				.attr("x", function(d, i) {
					return i * ls_w - ls_w;
					// return 0;
				})
				.attr("y", 550)
				.attr("width", ls_w)
				.attr("height", ls_h)
				.style("fill", function(d, i) {
					return color(d);
				})
				.style("opacity", 0.8);

			legend
				.append("text")
				.attr("x", function(d, i) {
					return i * ls_w - ls_w;
					// return 0;
				})
				.attr("y", 590)
				.text(function(d, i) {
					return legend_labels[i];
				});

			var legend_title = "Number of ICU beds Available";

			svg.append("text")
				// .attr("x", 600)
				.attr("x", 0)
				.attr("y", 540)
				.attr("class", "legend_title")
				.text(function() {
					return legend_title;
				});
		},
	},
};
</script>

<style lang="scss">
.heatmap {
	path {
		// stroke: white;
		// stroke-width: 1px;
	}

	body {
		font-family: "Proxima Nova", sans-serif;
	}

	.county {
		font: 14px sans-serif;
		font-weight: bold;
		fill: #fff;
	}
	// .states {
	// 	stroke: red;
	// 	stroke-width: 2px;
	// }
	.states {
		fill: none;
		stroke: white;
		stroke-width: 1px;
		stroke-linejoin: round;
	}

	.legend {
		font-size: 14px;
		font-family: "Proxima Nova", sans-serif;
	}

	.legend_title {
		font-size: 14px;
		font-family: "Proxima Nova", sans-serif;
		font-weight: bold;
	}

	div.tooltip {
		position: absolute;
		left: 75px;
		text-align: center;
		height: 16px;
		padding: 10px;
		font-size: 14px;
		background: #ffffff;
		border: 1px solid #989898;
		pointer-events: none;
	}

	p {
		font-family: "Proxima Nova", sans-serif;
		font-size: 10px;
		margin: 20px 0 0 10px;
	}

	@media (max-width: 400px) {
		.d3map {
			display: none;
		}
	}
}
</style>
