<template>
	<el-tabs type="border-card">
		<el-tab-pane label="ClubElo">
			<el-divider content-position="left">API 列表</el-divider>
			<el-card style="max-width: 480px">
				<template #header>
					<div class="card-header">
						<span style="font-size: 20px; font-weight: 600"
							>property seasons: List[str]</span
						>
						<br />
						<br />
						<span>Return a list of selected seasons.</span>
					</div>
				</template>
				<div>
					<el-form ref="seasonForm" :model="formModel">
						<el-form-item label="proxy">
							<el-input
								v-model="formModel.proxy"
								placeholder="代理地址"
								clearable
							></el-input>
						</el-form-item>
						<el-form-item label="no_cache">
							<el-switch v-model="formModel.no_cache"></el-switch>
						</el-form-item>
						<el-form-item label="no_store">
							<el-switch v-model="formModel.no_store"></el-switch>
						</el-form-item>
						<el-form-item label="data_dir">
							<el-input
								v-model="formModel.data_dir"
								placeholder="文件存储地址"
								clearable
							></el-input>
						</el-form-item>
					</el-form>
				</div>
				<template #footer>
					<el-button type="primary">发送请求</el-button>
				</template>
			</el-card>
			<el-divider content-position="left">返回结果</el-divider>
		</el-tab-pane>
		<el-tab-pane label="ESPN" disabled>ESPN</el-tab-pane>
		<el-tab-pane label="FBref" disabled>FBref</el-tab-pane>
		<el-tab-pane label="FotMob" disabled>FotMob</el-tab-pane>
		<el-tab-pane label="MatchHistory" disabled>MatchHistory</el-tab-pane>
		<el-tab-pane label="SoFIFA" disabled>SoFIFA</el-tab-pane>
		<el-tab-pane label="Understat" disabled>Understat</el-tab-pane>
	</el-tabs>
</template>
<script lang="ts" setup>
import { reactive } from "vue";
const formModel = reactive({
	proxy: "",
	no_cache: false,
	no_store: false,
	data_dir: ""
});
</script>

<style>
.demo-form-inline .el-input {
	--el-input-width: 220px;
}

.demo-form-inline .el-select {
	--el-select-width: 220px;
}
</style>
