<template>
	<div class="county-heatmap">
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
