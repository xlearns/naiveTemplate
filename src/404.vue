<template>
	<n-result
		status="404"
		title="404 资源不存在"
		description="虽然你不可能错，但我们确实没找到"
	/>
</template>
<script lang="ts">
import { defineComponent } from "vue";
import { NResult } from "naive-ui";
export default defineComponent({
	components: {
		NResult,
	},
	setup() {},
});
</script>
