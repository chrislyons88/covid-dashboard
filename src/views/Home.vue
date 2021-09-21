<template>
	<div class="home">
		<CountyHeatmap />
	</div>
</template>

<script>
// @ is an alias to /src

import CountyHeatmap from "@/components/CountyHeatmap.vue";

export default {
	name: "Home",
	components: {
		CountyHeatmap,
	},
};
</script>
