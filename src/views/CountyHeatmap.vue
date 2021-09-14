<template>
	<div class="county-heatmap">
		<div id="nav">
			<router-link to="/">View US county-level metrics</router-link>
			<router-link to="/states">View US state-level metrics</router-link>
		</div>

		<CountyHeatmap />
	</div>
</template>

<script>
// @ is an alias to /src

import CountyHeatmap from "@/components/CountyHeatmap.vue";

export default {
	name: "County Heatmap",
	components: {
		CountyHeatmap,
	},
};
</script>
