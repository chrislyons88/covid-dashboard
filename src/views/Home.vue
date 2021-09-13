<template>
	<div class="home">
		<router-link to="/states">View state level stats</router-link>
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
