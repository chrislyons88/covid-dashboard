<template>
	<div id="nav">
		<router-link to="/">ICU Beds per US County</router-link>
		<router-link to="/states">US State Infection Rates Over Time</router-link>
		<router-link to="/county-metrics"
			>US County metrics datagrid</router-link
		>
		<router-link to="/sankey">Sankey Diagram of Vaccine Data</router-link>
	</div>
</template>

<script>
export default {
	name: "Navigation",
};
</script>

<style scoped></style>
