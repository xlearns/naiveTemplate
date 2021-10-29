<template>
	<n-config-provider :locale="zc" :date-locale="dl">
		<n-message-provider>
			<router-view />
		</n-message-provider>
	</n-config-provider>
</template>

<script lang="ts">
import { defineComponent, reactive, toRefs } from "vue";
import {
	darkTheme,
	zhCN,
	dateZhCN,
	NConfigProvider,
	NMessageProvider,
} from "naive-ui";
export default defineComponent({
	components: {
		darkTheme,
		zhCN,
		dateZhCN,
		NConfigProvider,
		NMessageProvider,
	},
	setup() {
		const state = reactive({
			dt: darkTheme,
			zc: zhCN,
			dl: dateZhCN,
		});
		return {
			...toRefs(state),
		};
	},
});
</script>

<style lang="scss">
html,
body {
	width: 100%;
	height: 100%;
}
#app {
	position: relative;
	width: 100%;
	height: 100%;
	.n-config-provider {
		width: 100%;
		height: 100%;
	}
}
</style>
