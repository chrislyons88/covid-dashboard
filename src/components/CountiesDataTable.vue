<template>
	<div>
		<h1>Covid Metrics by US County</h1><br>
		<!-- <ul v-for="county in covidData" v-bind:key="county.fips">
			<li>{{ county.county }}</li>
			<p>{{ county.state }}</p>
		</ul> -->
		<div v-if="!covidData.length">
			<Spinner />
		</div>
		<ag-grid-vue
			v-else
			style="width: 100%; height: 600px;"
			class="ag-theme-alpine"
			:columnDefs="columnDefs"
			:rowData="rowData"
			rowSelection="multiple"
		>
		</ag-grid-vue>
	</div>
</template>

<script>
import "ag-grid-community/dist/styles/ag-grid.css";
import "ag-grid-community/dist/styles/ag-theme-alpine.css";
import { AgGridVue } from "ag-grid-vue3";
import Spinner from "@/components/Spinner.vue";

export default {
	name: "CountiesDataTable",
	props: {},
	components: {
		AgGridVue,
		Spinner,
	},
	data() {
		return {
			apiBaseUrl: "https://api.covidactnow.org/v2/",
			apiKey: "55783d149c7b461b87d3df4de4ffa9a1",
			covidData: [],
			columnDefs: null,
			gridApi: null,
			// rowData: null,
		};
	},
	computed: {
		covidCountyDataUrl() {
			return this.apiBaseUrl + "counties.json?apiKey=" + this.apiKey;
		},
		rowData() {
			var rows = [];
			this.covidData.forEach(function(d) {
				rows.push({
					county: d.county,
					state: d.state,
					"vax completed ratio": d.metrics.vaccinationsCompletedRatio,
					"ICU capacity ratio": d.metrics.icuCapacityRatio,
					cases: d.actuals.cases,
					deaths: d.actuals.deaths,
					"test positivity ratio": d.metrics.testPositivityRatio,
				});
			});
			return rows;
		},
	},
	async created() {
		await this.getData();
	},
	beforeMount() {
		this.defaultColDef = {
			resizable: true,
		};

		this.columnDefs = [
			{
				field: "county",
				sortable: true,
				filter: true,
				checkboxSelection: true,
			},
			{ field: "state", sortable: true, filter: true },
			{ field: "vax completed ratio", sortable: true, filter: true },
			{ field: "ICU capacity ratio", sortable: true, filter: true },
			{ field: "cases", sortable: true, filter: true },
			{ field: "deaths", sortable: true, filter: true },
			{ field: "test positivity ratio", sortable: true, filter: true },
		];
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
	},
};
</script>

<style lang="scss"></style>
