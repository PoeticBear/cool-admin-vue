<template>
	<cl-crud ref="Crud">
		<cl-row>
			<cl-refresh-btn></cl-refresh-btn>
			<cl-add-btn />
			<cl-multi-delete-btn />

			<el-button type="primary" @click="handleSyncFromSD">从 soccerdata 同步</el-button>
			<cl-flex1 />
			<cl-search-key placeholder="搜索名称" />
		</cl-row>

		<cl-row>
			<cl-table ref="Table" />
		</cl-row>

		<cl-row>
			<cl-flex1 />
			<cl-pagination />
		</cl-row>

		<cl-upsert ref="Upsert" />
	</cl-crud>

	<el-dialog v-model="syncDialogVisible" title="从 soccerdata 同步" width="1200">
		<el-tabs>
			<el-tab-pane label="FotMob">
				<el-card>
					<el-form
						ref="syncFormModel"
						:label-position="labelPosition"
						label-width="auto"
						inline
					>
						<el-form-item label="proxy">
							<el-input></el-input>
						</el-form-item>
						<el-form-item label="no_cache">
							<el-switch v-model="no_cache"></el-switch>
						</el-form-item>
						<el-form-item label="no_store">
							<el-switch v-model="no_store"></el-switch>
						</el-form-item>
						<el-form-item label="data_dir">
							<el-input></el-input>
						</el-form-item>
					</el-form>

					<template #footer>
						<el-button type="primary" @click="syncFromSoccerData">获取</el-button>
					</template>
				</el-card>
				<el-divider content-position="left">返回结果(json)</el-divider>
				<cl-editor-monaco
					:modelValue="scrapeList"
					height="300"
					language="typescript"
				></cl-editor-monaco>
				<el-divider content-position="left">返回结果(表格)</el-divider>
				<el-table :data="scrapedInTable" fit>
					<el-table-column prop="league" label="联赛名称"></el-table-column>
					<el-table-column prop="season" label="赛季"></el-table-column>
					<el-table-column prop="season_id" label="赛季年份"></el-table-column>
					<el-table-column prop="url" label="联赛赛季主页">
						<template #default="scope">
							<a :href="scope.row.url" target="_blank">{{ scope.row.url }}</a>
						</template>
					</el-table-column>
					<el-table-column prop="exists" label="已存在">
						<template #default="scope">
							<el-tag v-if="scope.row.exists === '已存在'" type="success">{{
								scope.row.exists
							}}</el-tag>
							<el-tag v-else type="danger">{{ scope.row.exists }}</el-tag>
						</template>
					</el-table-column>
					<el-table-column prop="op" label="操作">
						<template #default="scope">
							<el-button
								link
								type="primary"
								size="small"
								@click="handleSyncSingle(scope.row)"
								>同步</el-button
							>
						</template>
					</el-table-column>
				</el-table>
			</el-tab-pane>
		</el-tabs>
	</el-dialog>
</template>

<script lang="ts" name="soccer-player" setup>
import { useTable, useUpsert, useCrud } from "@cool-vue/crud";
import { useCool } from "/@/cool";
import { ref } from "vue";
import type { FormProps } from "element-plus";

type FreekickPlayer = {};

const { service } = useCool();
const syncDialogVisible = ref(false);
const no_cache = ref(false);
const no_store = ref(false);
let scrapeList = ref<FreekickPlayer[]>([]);
const labelPosition = ref<FormProps["labelPosition"]>("right");
const existList = ref<FreekickPlayer[]>([]);
const scrapedInTable = ref([]);
const Crud = useCrud(
	{
		service: service.soccer.player,
		async onRefresh(params, { next }) {
			const res = await next(params);
			console.log("获取到的列表:", res);
			existList.value = res.list;
		}
	},
	(app) => {
		app.refresh();
	}
);

console.log("Crud", Crud);

const Upsert = useUpsert({
	dialog: {
		width: "800px"
	},
	items: [
		{
			label: "球员名称",
			span: 24,
			prop: "name",
			rules: [{ required: true, message: "请输入球员名称", trigger: "blur" }],
			component: {
				name: "el-input"
			}
		},
		{
			label: "所属球队",
			span: 24,
			prop: "soccer_team_id",
			rules: [{ required: false, message: "请选择所属球队", trigger: "blur" }],
			component: {
				name: "el-select"
			}
		},
		{
			label: "出生日期",
			span: 24,
			prop: "date",
			rules: [{ required: false, message: "请选择出生日期", trigger: "blur" }],
			component: {
				name: "el-date-picker"
			}
		},
		{
			label: "国籍",
			span: 24,
			prop: "nationality",
			rules: [{ required: false, message: "请输入国籍", trigger: "blur" }],
			component: {
				name: "el-input"
			}
		},
		{
			label: "场上位置",
			span: 24,
			prop: "position",
			rules: [{ required: false, message: "请输入场上位置", trigger: "blur" }],
			component: {
				name: "el-input"
			}
		},
		{
			label: "身高",
			span: 24,
			prop: "height",
			rules: [{ required: false, message: "请输入身高", trigger: "blur" }],
			component: {
				name: "el-input"
			}
		},
		{
			label: "体重",
			span: 24,
			prop: "weight",
			rules: [{ required: false, message: "请输入体重", trigger: "blur" }],
			component: {
				name: "el-input"
			}
		},
		{
			label: "身价",
			span: 24,
			prop: "marketValue",
			rules: [{ required: false, message: "请输入身价", trigger: "blur" }],
			component: {
				name: "el-input"
			}
		}
	]
});

const Table = useTable({
	columns: [
		{
			prop: "name",
			label: "球员全名",
			minWidth: 100
		},
		{
			prop: "soccer_team_id",
			label: "所属球队",
			minWidth: 100
		},
		{
			prop: "date",
			label: "出生日期",
			minWidth: 100
		},
		{
			prop: "nationality",
			label: "国籍",
			minWidth: 100
		},
		{
			prop: "position",
			label: "场上位置",
			minWidth: 100
		},
		{
			prop: "height",
			label: "身高（单位：厘米）",
			minWidth: 100
		},
		{
			prop: "weight",
			label: "体重（单位：千克）",
			minWidth: 100
		},
		{
			prop: "marketValue",
			label: "身价（单位：欧元）",
			minWidth: 100
		}
	]
});

const handleSyncFromSD = () => {
	syncDialogVisible.value = true;
};

const syncFromSoccerData = () => {
	scrapeList.value = [];

	const result = markDataAsExisting(existList.value, scrapeList.value);
	scrapedInTable.value = result;
	console.log("result", result);

	// service
	// 	.request({
	// 		url: "http://127.0.0.1:5000/api/fotmob/readSeasons",
	// 		method: "GET",
	// 		data: {},
	// 		params: {},
	// 		header: {
	// 			token: ""
	// 		}
	// 	})
	// 	.then((res) => {
	// 		responsetList.value = res.data;
	// 		console.log(res);
	// 	});
};

const handleSyncSingle = (row: FreekickPlayer) => {
	service.soccer.player.addPlayer(row).then((res) => {
		console.log(res);
	});
};

const markDataAsExisting = (existList, scrapeList) => {
	const markedData = scrapeList.map((scrapeItem) => {
		const existItem = existList.find(
			(data) => data.league === scrapeItem.league && data.country === scrapeItem.country
		);
		scrapeItem.exists = existItem ? "已存在" : "新增数据";
		return scrapeItem;
	});
	return markedData;
};
</script>
