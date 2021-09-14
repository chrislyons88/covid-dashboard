<template>
	<div class="states">
		<div id="nav">
			<router-link to="/">View US county-level metrics</router-link>
			<router-link to="/county-heatmap"
				>View US county heatmap</router-link
			>
		</div>

		<StateTimeseries />
	</div>
</template>

<script>
// @ is an alias to /src

import StateTimeseries from "@/components/StateTimeseries.vue";

export default {
	name: "States",
	components: {
		StateTimeseries,
	},
};
</script>
