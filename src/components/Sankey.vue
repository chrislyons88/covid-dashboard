<template>
	<div class="heatmap">
		<h1>Sankey Diagram of Available Vaccination Data</h1>
		<div v-if="!covidData.length">
			<Spinner />
		</div>
		<div v-else>
			<div class="chart">
				<canvas id="chart"></canvas>
			</div>
			<footer>
				<p>Data provided by the <a target="_blank" href="https://apidocs.covidactnow.org/">Covid Act Now API</a></p>
			</footer>
		</div>
	</div>
</template>

<script>
import Spinner from "@/components/Spinner.vue";
import { Chart, LinearScale } from "chart.js";
import { SankeyController, Flow } from "chartjs-chart-sankey";

Chart.register(LinearScale, SankeyController, Flow);

export default {
	components: {
		Spinner,
	},
	data: function() {
		return {
			apiBaseUrl: "https://api.covidactnow.org/v2/",
			apiKey: "55783d149c7b461b87d3df4de4ffa9a1",
			covidData: [],
			// sortedData: {},
			sankeyDiagram: {},
			sankeyColors: {
				"Vaccinations Distributed": "black",
				"Vaccinations Unused": "orange",
				"Vaccinations Administered": "gray",
				"Vaccinations Initiated": "blue",
				"Vaccinations Completed": "green",
				"12-14": "red",
				"15-24": "green",
				"25-34": "blue",
				"35-44": "indigo",
				"45-54": "violet",
				"55-64": "orange",
				"65-74": "yellow",
				"75-84": "purple",
				"85_plus": "brown",
			},
			// chartData: [
			// 	{
			// 		from: "Vacinations Distributed",
			// 		to: "Vaccinations Administered",
			// 		// flow: this.sortedData.distributed,
			// 		flow: 10000000,
			// 	},
			// 	{
			// 		from: "Vacinations Distributed",
			// 		to: "Vaccinations Unused",
			// 		flow: 80000,
			// 	},
			// 	{
			// 		from: "Vaccinations Administered",
			// 		to: "Vaccinations Initiated",
			// 		flow: 500000,
			// 	},
			// 	{
			// 		from: "Vaccinations Administered",
			// 		to: "Vaccinations Commpleted",
			// 		flow: 500000,
			// 	},
			// ],
		};
	},
	computed: {
		covidCountyDataUrl() {
			return this.apiBaseUrl + "states.json?apiKey=" + this.apiKey;
		},
		unused() {
			return this.sortedData.distributed - this.sortedData.administered;
		},
		chartData() {
			return [
				{
					from: "Vacinations Distributed",
					to: "Vaccinations Administered",
					flow: this.sortedData.administered,
					// flow: 10000000,
				},
				{
					from: "Vacinations Distributed",
					to: "Vaccinations Unused",
					flow: this.unused,
				},
				{
					from: "Vaccinations Administered",
					to: "Vaccinations Initiated",
					flow: this.sortedData.initiated,
				},
				{
					from: "Vaccinations Administered",
					to: "Vaccinations Completed",
					flow: this.sortedData.completed,
				},

				{
					from: "Vaccinations Initiated",
					to: "12-14",
					flow: this.sortedData.demographicsAges["12-14"],
				},
				{
					from: "Vaccinations Initiated",
					to: "15-24",
					flow: this.sortedData.demographicsAges["15-24"],
				},
				{
					from: "Vaccinations Initiated",
					to: "25-34",
					flow: this.sortedData.demographicsAges["25-34"],
				},
				{
					from: "Vaccinations Initiated",
					to: "35-44",
					flow: this.sortedData.demographicsAges["35-44"],
				},
				{
					from: "Vaccinations Initiated",
					to: "45-54",
					flow: this.sortedData.demographicsAges["45-54"],
				},
				{
					from: "Vaccinations Initiated",
					to: "55-64",
					flow: this.sortedData.demographicsAges["55-64"],
				},
				{
					from: "Vaccinations Initiated",
					to: "65-74",
					flow: this.sortedData.demographicsAges["65-74"],
				},
				{
					from: "Vaccinations Initiated",
					to: "75-84",
					flow: this.sortedData.demographicsAges["75-84"],
				},
				{
					from: "Vaccinations Initiated",
					to: "85_plus",
					flow: this.sortedData.demographicsAges["85_plus"],
				},

			]
		},
		sortedData() {
			let that = this;
			let distributed = 0;
			let unused = 0;
			let administered = 0;
			let initiated = 0;
			let completed = 0;
			let demographicsAges = {
				"12-14": 0,
				"15-24": 0,
				"25-34": 0,
				"35-44": 0,
				"45-54": 0,
				"55-64": 0,
				"65-74": 0,
				"75-84": 0,
				"85_plus": 0,
			};
			let sex = { female: 0, male: 0, unknown: 0 };

			this.covidData.forEach(function(d) {
				distributed += !isNaN(d.actuals.vaccinesDistributed)
					? d.actuals.vaccinesDistributed
					: 0;
				initiated += !isNaN(d.actuals.vaccinationsInitiated)
					? d.actuals.vaccinationsInitiated
					: 0;
				completed += !isNaN(d.actuals.vaccinationsCompleted)
					? d.actuals.vaccinationsCompleted
					: 0;
				administered += !isNaN(d.actuals.vaccinesAdministered)
					? d.actuals.vaccinesAdministered
					: 0;
				if (d.actuals.vaccinationsInitiatedDemographics != null) {
					demographicsAges["12-14"] += d.actuals
						.vaccinationsInitiatedDemographics.age["12-14"]
						? d.actuals.vaccinationsInitiatedDemographics.age[
								"12-14"
						]
						: 0;
					demographicsAges["15-24"] += d.actuals
						.vaccinationsInitiatedDemographics.age["15-24"]
						? d.actuals.vaccinationsInitiatedDemographics.age[
								"15-24"
						]
						: 0;
					demographicsAges["25-34"] += d.actuals
						.vaccinationsInitiatedDemographics.age["25-34"]
						? d.actuals.vaccinationsInitiatedDemographics.age[
								"25-34"
						]
						: 0;
					demographicsAges["35-44"] += d.actuals
						.vaccinationsInitiatedDemographics.age["35-44"]
						? d.actuals.vaccinationsInitiatedDemographics.age[
								"35-44"
						]
						: 0;
					demographicsAges["45-54"] += d.actuals
						.vaccinationsInitiatedDemographics.age["45-54"]
						? d.actuals.vaccinationsInitiatedDemographics.age[
								"45-54"
						]
						: 0;
					demographicsAges["55-64"] += d.actuals
						.vaccinationsInitiatedDemographics.age["55-64"]
						? (demographicsAges["55-64"] +=
								d.actuals.vaccinationsInitiatedDemographics.age[
									"55-64"
								])
						: 0;
					demographicsAges["65-74"] += d.actuals
						.vaccinationsInitiatedDemographics.age["65-74"]
						? (demographicsAges["65-74"] +=
								d.actuals.vaccinationsInitiatedDemographics.age[
									"65-74"
								])
						: 0;
					demographicsAges["75-84"] += d.actuals
						.vaccinationsInitiatedDemographics.age["75-84"]
						? d.actuals.vaccinationsInitiatedDemographics.age[
								"75-84"
						]
						: 0;
					demographicsAges["85_plus"] += d.actuals
						.vaccinationsInitiatedDemographics.age["85_plus"]
						? d.actuals.vaccinationsInitiatedDemographics.age[
								"85_plus"
						]
						: 0;
				}
				if (d.actuals.vaccinationsInitiatedDemographics != null) {
					sex["male"] += d.actuals.vaccinationsInitiatedDemographics
						.sex["male"]
						? d.actuals.vaccinationsInitiatedDemographics.sex[
								"male"
						]
						: 0;
					sex["female"] += d.actuals.vaccinationsInitiatedDemographics
						.sex["female"]
						? d.actuals.vaccinationsInitiatedDemographics.sex[
								"female"
						]
						: 0;
					sex["unknown"] += d.actuals
						.vaccinationsInitiatedDemographics.sex["unknown"]
						? d.actuals.vaccinationsInitiatedDemographics.sex[
								"unknown"
						]
						: 0;
				}
			});

			// console.log(distributed);

			return {
				distributed: distributed,
				initiated: initiated,
				completed: completed,
				administered: administered,
				demographicsAges: demographicsAges,
				sex: sex,
			};
		}
	},
	async created() {
		await this.getData();
		// await this.populateSortedData();
		await this.drawSankey();
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

		populateSortedData() {
			let that = this;
			let distributed = 0;
			let unused = 0;
			let administered = 0;
			let initiated = 0;
			let completed = 0;
			let demographicsAges = {
				"12-14": 0,
				"15-24": 0,
				"25-34": 0,
				"35-44": 0,
				"45-54": 0,
				"55-64": 0,
				"65-74": 0,
				"75-84": 0,
				"85_plus": 0,
			};
			let sex = { female: 0, male: 0, unknown: 0 };

			this.covidData.forEach(function(d) {
				distributed += !isNaN(d.actuals.vaccinesDistributed)
					? d.actuals.vaccinesDistributed
					: 0;
				initiated += !isNaN(d.actuals.vaccinationsInitiated)
					? d.actuals.vaccinationsInitiated
					: 0;
				completed += !isNaN(d.actuals.vaccinationsCompleted)
					? d.actuals.vaccinationsCompleted
					: 0;
				administered += !isNaN(d.actuals.vaccinesAdministered)
					? d.actuals.vaccinesAdministered
					: 0;
				if (d.actuals.vaccinationsInitiatedDemographics != null) {
					demographicsAges["12-14"] += d.actuals
						.vaccinationsInitiatedDemographics.age["12-14"]
						? d.actuals.vaccinationsInitiatedDemographics.age[
								"12-14"
						]
						: 0;
					demographicsAges["15-24"] += d.actuals
						.vaccinationsInitiatedDemographics.age["15-24"]
						? d.actuals.vaccinationsInitiatedDemographics.age[
								"15-24"
						]
						: 0;
					demographicsAges["25-34"] += d.actuals
						.vaccinationsInitiatedDemographics.age["25-34"]
						? d.actuals.vaccinationsInitiatedDemographics.age[
								"25-34"
						]
						: 0;
					demographicsAges["35-44"] += d.actuals
						.vaccinationsInitiatedDemographics.age["35-44"]
						? d.actuals.vaccinationsInitiatedDemographics.age[
								"35-44"
						]
						: 0;
					demographicsAges["45-54"] += d.actuals
						.vaccinationsInitiatedDemographics.age["45-54"]
						? d.actuals.vaccinationsInitiatedDemographics.age[
								"45-54"
						]
						: 0;
					demographicsAges["55-64"] += d.actuals
						.vaccinationsInitiatedDemographics.age["55-64"]
						? (demographicsAges["55-64"] +=
								d.actuals.vaccinationsInitiatedDemographics.age[
									"55-64"
								])
						: 0;
					demographicsAges["65-74"] += d.actuals
						.vaccinationsInitiatedDemographics.age["65-74"]
						? (demographicsAges["65-74"] +=
								d.actuals.vaccinationsInitiatedDemographics.age[
									"65-74"
								])
						: 0;
					demographicsAges["75-84"] += d.actuals
						.vaccinationsInitiatedDemographics.age["75-84"]
						? d.actuals.vaccinationsInitiatedDemographics.age[
								"75-84"
						]
						: 0;
					demographicsAges["85_plus"] += d.actuals
						.vaccinationsInitiatedDemographics.age["85_plus"]
						? d.actuals.vaccinationsInitiatedDemographics.age[
								"85_plus"
						]
						: 0;
				}
				if (d.actuals.vaccinationsInitiatedDemographics != null) {
					sex["male"] += d.actuals.vaccinationsInitiatedDemographics
						.sex["male"]
						? d.actuals.vaccinationsInitiatedDemographics.sex[
								"male"
						]
						: 0;
					sex["female"] += d.actuals.vaccinationsInitiatedDemographics
						.sex["female"]
						? d.actuals.vaccinationsInitiatedDemographics.sex[
								"female"
						]
						: 0;
					sex["unknown"] += d.actuals
						.vaccinationsInitiatedDemographics.sex["unknown"]
						? d.actuals.vaccinationsInitiatedDemographics.sex[
								"unknown"
						]
						: 0;
				}
			});

			// console.log(distributed);

			this.sortedData = {
				distributed: distributed,
				initiated: initiated,
				completed: completed,
				administered: administered,
				demographicsAges: demographicsAges,
				sex: sex,
			};
		},
		drawSankey() {
			let that = this;
			var ctx = document.getElementById("chart").getContext("2d");

			// var colors = {
			// 	Oil: "black",
			// 	Coal: "gray",
			// 	"Fossil Fuels": "slategray",
			// 	Electricity: "blue",
			// 	Energy: "orange",
			// };

			function getColor(name) {
				return that.sankeyColors[name] || "black";
			}

			this.sankeyDiagram = new Chart(ctx, {
				type: "sankey",
				data: {
					datasets: [
						{
							data: that.chartData,
							colorFrom: (c) =>
								getColor(c.dataset.data[c.dataIndex].from),
							colorTo: (c) =>
								getColor(c.dataset.data[c.dataIndex].to),
						},
					],
				},
			});

			// end drawsankey
		},
	},
};
</script>

<style lang="scss"></style>
