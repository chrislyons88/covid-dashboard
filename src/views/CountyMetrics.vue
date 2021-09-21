<template>
	<div class="county-metrics">
		<CountiesDataTable />
	</div>
</template>

<script>
// @ is an alias to /src

import CountiesDataTable from "@/components/CountiesDataTable.vue";

export default {
	name: "CountyMetrics",
	components: {
		CountiesDataTable,
	},
};
</script>
