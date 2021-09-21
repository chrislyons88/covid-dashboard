<template>
	<div class="sankey">
		<Sankey />
	</div>
</template>

<script>
// @ is an alias to /src

import Sankey from "@/components/Sankey.vue";

export default {
	name: "County Heatmap",
	components: {
		Sankey,
	},
};
</script>
