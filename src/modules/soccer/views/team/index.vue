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

<script lang="ts" name="soccer-season" setup>
import { useTable, useUpsert, useCrud } from "@cool-vue/crud";
import { useCool } from "/@/cool";
import { reactive, ref } from "vue";
import type { FormProps } from "element-plus";

type FreekickTeam = {
	leagueId: string;
	name: string;
	nameCn: string;
	abbreviation: string;
	foundedYear: string;
	homeStadium: string;
};

const { service } = useCool();
const syncDialogVisible = ref(false);
const no_cache = ref(false);
const no_store = ref(false);
let scrapeList = ref<FreekickTeam[]>([]);
const labelPosition = ref<FormProps["labelPosition"]>("right");
const existList = ref<FreekickTeam[]>([]);
const scrapedInTable = ref([]);
const Crud = useCrud(
	{
		service: service.soccer.team,
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
			label: "所处联赛",
			span: 24,
			prop: "leagueId",
			rules: [{ required: true, message: "请选择联赛", trigger: "blur" }],
			component: {
				name: "el-select",
				options: [
					{ label: "英超", value: "ENG-Premier League" },
					{ label: "西甲", value: "ESP-La Liga" },
					{ label: "法甲", value: "FRA-Ligue 1" },
					{ label: "德甲", value: "GER-Bundesliga" },
					{ label: "意甲", value: "ITA-Serie A" }
				]
			}
		},
		{
			label: "球队名称",
			span: 24,
			prop: "name",
			rules: [{ required: false, message: "请输入球队名称", trigger: "blur" }],
			component: {
				name: "el-input"
			}
		},
		{
			label: "中文名称",
			span: 24,
			prop: "nameCn",
			rules: [{ required: false, message: "请输入中文名称", trigger: "blur" }],
			component: {
				name: "el-input"
			}
		},
		{
			label: "球队简称",
			span: 24,
			prop: "abbreviation",
			rules: [{ required: false, message: "请输入球队简称", trigger: "blur" }],
			component: {
				name: "el-input"
			}
		},
		{
			label: "建队年份",
			span: 24,
			prop: "foundedYear",
			rules: [{ required: false, message: "请输入建队年份", trigger: "blur" }],
			component: {
				name: "el-input"
			}
		},
		{
			label: "球队主场",
			span: 24,
			prop: "homeStadium",
			rules: [{ required: false, message: "请输入球队主场", trigger: "blur" }],
			component: {
				name: "el-input"
			}
		}
	]
});

const Table = useTable({
	columns: [
		{
			type: "selection",
			minWidth: 55
		},
		{
			prop: "leagueId",
			label: "联赛ID",
			minWidth: 100
		},
		{
			prop: "name",
			label: "球队名称",
			minWidth: 100
		},
		{
			prop: "nameCn",
			label: "球队中文名称",
			minWidth: 100
		},
		{
			prop: "abbreviation",
			label: "球队缩写",
			minWidth: 100
		},
		{
			prop: "foundedYear",
			label: "成立年份",
			minWidth: 100
		},
		{
			prop: "homeStadium",
			label: "主场球场",
			minWidth: 100
		},
		{
			prop: "createTime",
			label: "创建时间",
			minWidth: 100
		},
		{
			prop: "updateTime",
			label: "更新时间",
			minWidth: 100
		},
		{
			label: "操作",
			type: "op",
			minWidth: 100,
			buttons: ["edit", "delete"]
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

const handleSyncSingle = (row: FreekickTeam) => {
	service.soccer.team.addTeam(row).then((res) => {
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
