<template>
	<div class="home">
		<div id="nav">
			<router-link to="/states">View US state-level metrics</router-link>
			<router-link to="/county-heatmap"
				>View US county heatmap</router-link
			>
		</div>

		<CountiesDataTable />
	</div>
</template>

<script>
// @ is an alias to /src

import CountiesDataTable from "@/components/CountiesDataTable.vue";

export default {
	name: "Home",
	components: {
		CountiesDataTable,
	},
};
</script>
