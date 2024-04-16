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

type FreekickSeason = {
	league: string;
	league_id: number;
	season: string;
	season_id: string;
	url: string;
};

const { service } = useCool();
const syncDialogVisible = ref(false);
const no_cache = ref(false);
const no_store = ref(false);
let scrapeList = ref<FreekickSeason[]>([]);
const labelPosition = ref<FormProps["labelPosition"]>("right");
const existList = ref<FreekickSeason[]>([]);
const scrapedInTable = ref([]);
const Crud = useCrud(
	{
		service: service.soccer.season,
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
			label: "联赛名称",
			span: 24,
			prop: "league",
			rules: [{ required: true, message: "请输入联赛名称", trigger: "blur" }],
			component: {
				name: "el-input"
			}
		},
		{
			label: "赛季名称",
			span: 24,
			prop: "season",
			rules: [{ required: false, message: "请输入赛季名称", trigger: "blur" }],
			component: {
				name: "el-input"
			}
		},
		{
			label: "赛季id",
			span: 24,
			prop: "season_id",
			rules: [{ required: false, message: "请输入赛季id", trigger: "blur" }],
			component: {
				name: "el-input"
			}
		},
		{
			label: "联赛赛季主页",
			span: 24,
			prop: "url",
			rules: [{ required: false, message: "请输入联赛赛季主页", trigger: "blur" }],
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
			prop: "league",
			label: "联赛名称",
			minWidth: 100
		},
		{
			prop: "season",
			label: "赛季名称",
			minWidth: 100
		},
		{
			prop: "season_id",
			label: "赛季年份",
			minWidth: 100
		},
		{
			prop: "url",
			label: "联赛赛季主页",
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
	scrapeList.value = [
		{
			league: "ENG-Premier League",
			league_id: 47,
			season: "2324",
			season_id: "2023/2024",
			url: "https://fotmob.com/leagues/47/overview/premier-league?season=2023/2024"
		},
		{
			league: "ESP-La Liga",
			league_id: 87,
			season: "2324",
			season_id: "2023/2024",
			url: "https://fotmob.com/leagues/87/overview/laliga?season=2023/2024"
		},
		{
			league: "FRA-Ligue 1",
			league_id: 53,
			season: "2324",
			season_id: "2023/2024",
			url: "https://fotmob.com/leagues/53/overview/ligue-1?season=2023/2024"
		},
		{
			league: "GER-Bundesliga",
			league_id: 54,
			season: "2324",
			season_id: "2023/2024",
			url: "https://fotmob.com/leagues/54/overview/bundesliga?season=2023/2024"
		},
		{
			league: "INT-Women's World Cup",
			league_id: 76,
			season: "2324",
			season_id: "2023",
			url: "https://fotmob.com/leagues/76/overview/womens-world-cup?season=2023"
		},
		{
			league: "ITA-Serie A",
			league_id: 55,
			season: "2324",
			season_id: "2023/2024",
			url: "https://fotmob.com/leagues/55/overview/serie?season=2023/2024"
		}
	];

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

const handleSyncSingle = (row: FreekickSeason) => {
	console.log(row.league + ":" + row.season);
	service.soccer.season.addSeason(row).then((res) => {
		console.log(res);
	});
};

const markDataAsExisting = (existList, scrapeList) => {
	const markedData = scrapeList.map((scrapeItem) => {
		const existItem = existList.find(
			(data) => data.league === scrapeItem.league && data.season_id === scrapeItem.season_id
		);
		scrapeItem.exists = existItem ? "已存在" : "新增数据";
		return scrapeItem;
	});
	return markedData;
};
</script>
