<template>
	<div class="states">
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
