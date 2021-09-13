<template>
	<div>
		<h1>Covid infection rates by state</h1>
		<div v-if="!covidData.length">
			<Spinner />
		</div>
		<canvas v-else id="timeseries-chart"></canvas>
	</div>
</template>

<script>
import Chart from "chart.js/auto";
import colorLib from "@kurkle/color";
import Spinner from "@/components/Spinner.vue";

export default {
	name: "StateTimeSeries",
	props: {},
	components: {
		Spinner,
	},
	data() {
		return {
			apiBaseUrl: "https://api.covidactnow.org/v2/",
			apiKey: "55783d149c7b461b87d3df4de4ffa9a1",
			covidData: [],
			chartData: {
				type: "line",
				data: {
					labels: this.filteredLabels,
					datasets: [],
				},

				options: {
					responsive: true,
					
					interaction: {
						mode: "index",
						intersect: false,
					},
					stacked: false,
					plugins: {
						title: {
							display: true,
							text: " ",
						},
						tooltip: {
						mode: "nearest",
					},
					},
					scales: {
						y: {
							type: "linear",
							display: true,
							position: "left",
						},
						y1: {
							type: "linear",
							display: true,
							position: "right",

							// grid line settings
							grid: {
								drawOnChartArea: false, // only want the grid lines for one axis to show up
							},
						},
					},
				},
			},
		};
	},
	computed: {
		covidStateDataUrl() {
			return (
				this.apiBaseUrl + "states.timeseries.json?apiKey=" + this.apiKey
			);
		},
		filteredData() {
			var states = [];
			this.covidData.forEach(function(d) {
				states.push({
					state: d.state,
					metricsTimeseries: d.metricsTimeseries,
				});
			});
			return states;
		},
		filteredLabels() {
			let labels = [];
			let count = 0;
			this.filteredData.forEach(function(d) {
				if (count === 0) {
					d.metricsTimeseries.forEach(function(d) {
						labels.push(d.date);
					});
				}

				count++;
			});

			// labels = labels.slice(Math.max(labels.length - 12, 0));

			return labels;
		},
		filteredDatasets() {
			function transparentize(value, opacity) {
				var alpha = opacity === undefined ? 0.5 : 1 - opacity;
				return colorLib(value)
					.alpha(alpha)
					.rgbString();
			}
			let dataset = [
				// {
				// 	label: "Dataset 1",
				// 	data: [5, 4, 8, 6, 23, 7, 6],
				// 	borderColor: this.randomRgb(),
				// 	backgroundColor: this.transparentize(this.randomRgb(), 0.5),
				// },
				// {
				// 	label: "Dataset 2",
				// 	data: [1, 3, 5, 3, 6, 9, 6],
				// 	borderColor: this.randomRgb(),
				// 	backgroundColor: this.transparentize(this.randomRgb(), 0.5),
				// },
			];

			this.filteredData.forEach(function(d) {
				let dataObject = {};
				var o = Math.round,
					r = Math.random,
					s = 255;

				let randomColor =
					"rgb(" +
					o(r() * s) +
					"," +
					o(r() * s) +
					"," +
					o(r() * s) +
					")";

				dataObject.label = d.state;
				dataObject.borderColor = randomColor;
				dataObject.backgroundColor = transparentize(randomColor, 0.5);
				dataObject.data = [];

				d.metricsTimeseries.forEach(function(d) {
					dataObject.data.push(d.infectionRate);
				});

				dataset.push(dataObject);
			});

			// dataset = dataset.slice(Math.max(dataset.length - 12, 0));

			return dataset;
		},
	},
	async created() {
		await this.getData();
		await this.populateChartLabels();
		await this.populateDatasets();
		this.drawLineChart();
	},
	beforeMount() {},
	mounted() {},
	methods: {
		async getData() {
			try {
				let response = await fetch(this.covidStateDataUrl);
				this.covidData = await response.json();
			} catch (error) {
				console.log(error);
			}
		},
		randomRgb() {
			var o = Math.round,
				r = Math.random,
				s = 255;
			return (
				"rgb(" + o(r() * s) + "," + o(r() * s) + "," + o(r() * s) + ")"
			);
		},
		transparentize(value, opacity) {
			var alpha = opacity === undefined ? 0.5 : 1 - opacity;
			return colorLib(value)
				.alpha(alpha)
				.rgbString();
		},
		valueOrDefault(value, defaultValue) {
			return typeof value === "undefined" ? defaultValue : value;
		},
		drawLineChart() {
			const ctx = document.getElementById("timeseries-chart");
			new Chart(ctx, this.chartData);
		},
		populateChartLabels() {
			this.chartData.data.labels = this.filteredLabels;
			// this.chartData.data.labels = ["2021-2-3", "2021-2-3"];
		},
		populateDatasets() {
			this.chartData.data.datasets = this.filteredDatasets;
		},
	},
};
</script>

<style lang="scss">

</style>