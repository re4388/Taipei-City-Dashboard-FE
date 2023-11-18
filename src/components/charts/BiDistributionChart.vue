<!-- Developed by Taipei Urban Intelligence Center 2023 -->

<script setup>
import { ref, computed } from "vue";
import { useMapStore } from "../../store/mapStore";

const props = defineProps([
	"chart_config",
	"activeChart",
	"series",
	"map_config",
]);
const mapStore = useMapStore();

const seriesMinMax = computed(() => {
	const minMax = props.series.map((item) => {
		const values = item.data.map((dataItem) => {
			return dataItem.y;
		});
		return {
			min: Math.min(...values),
			max: Math.max(...values),
		};
	});
	return {
		min: Math.min(...minMax.map((item) => item.min)),
		max: Math.max(...minMax.map((item) => item.max)),
	};
});

const parsedSeries = computed(() => {
	return props.series.map((item) => {
		return {
			name: item.name,
			data: item.data.map((dataItem) => {
				const isPositive = dataItem.y > 0;
				let parsedY = null;
				let parsedGoal = null;

				if (isPositive) {
					parsedY = dataItem.y / seriesMinMax.value.max;
					parsedGoal = dataItem.marker / seriesMinMax.value.max;
				} else {
					parsedY = -(dataItem.y / seriesMinMax.value.min);
					parsedGoal = -(dataItem.marker / seriesMinMax.value.min);
				}

				const obj = {
					x: dataItem.x,
					y: parsedY.toFixed(5),
				};

				if (parsedGoal) {
					obj.goals = [
						{
							value: parsedGoal,
							strokeWidth: 3,
							strokeColor: "#775DD0",
						},
					];
				}

				return obj;
			}),
		};
	});
});

const chartOptions = ref({
	chart: {
		offsetY: 15,
		stacked: true,
		toolbar: {
			show: false,
		},
	},
	colors: props.chart_config.color,
	dataLabels: {
		offsetX: 20,
		distributed: true,
		textAnchor: "end",
		formatter: (val, { seriesIndex, dataPointIndex }) => {
			return Math.abs(
				props.series[seriesIndex].data[dataPointIndex].y
			).toString();
		},
	},
	grid: {
		show: false,
	},
	legend: {
		position: "top",
		show: props.series.length > 1 ? true : false,
		onItemClick: {
			toggleDataSeries: false,
		},
	},
	plotOptions: {
		bar: {
			borderRadius: 3,
			horizontal: true,
			borderRadiusApplication: "end",
			borderRadiusWhenStacked: "all",
		},
	},
	stroke: {
		colors: ["#CCC"],
		show: false,
		width: 1,
	},
	// The class "chart-tooltip" could be edited in /assets/styles/chartStyles.css
	tooltip: {
		custom: function ({ seriesIndex, dataPointIndex, w }) {
			return (
				'<div class="chart-tooltip">' +
				"<h6>" +
				w.globals.labels[dataPointIndex] +
				"</h6>" +
				"<span>" +
				`2022年：${Math.abs(
					props.series[seriesIndex].data[dataPointIndex].y
				)}` +
				` ${props.chart_config.unit}` +
				"</span>" +
				"</br>" +
				"<span>" +
				`2021年：${Math.abs(
					props.series[seriesIndex].data[dataPointIndex].marker
				)}` +
				` ${props.chart_config.unit}` +
				"</span>" +
				"</div>"
			);
		},
		followCursor: true,
	},
	xaxis: {
		axisBorder: {
			show: false,
		},
		axisTicks: {
			show: false,
		},
		labels: {
			show: false,
		},
		type: "category",
	},
	yaxis: {
		min: -1.2,
		max: 1.2,
	},
});

const chartHeight = computed(() => {
	const minHeight = 200;
	return props.series[0].data.length * 21 > minHeight
		? `${props.series[0].data.length * 21}`
		: `${minHeight}`;
});

const selectedIndex = ref(null);

function handleDataSelection(e, chartContext, config) {
	if (!props.chart_config.map_filter) {
		return;
	}
	if (config.dataPointIndex !== selectedIndex.value) {
		mapStore.addLayerFilter(
			`${props.map_config[0].index}-${props.map_config[0].type}`,
			props.chart_config.map_filter[0],
			props.chart_config.map_filter[1][config.dataPointIndex]
		);
		selectedIndex.value = config.dataPointIndex;
	} else {
		mapStore.clearLayerFilter(
			`${props.map_config[0].index}-${props.map_config[0].type}`
		);
		selectedIndex.value = null;
	}
}
</script>

<template>
	<div v-if="activeChart === 'BiDistributionChart'">
		<apexchart
			width="100%"
			:height="chartHeight"
			type="bar"
			:options="chartOptions"
			:series="parsedSeries"
			@dataPointSelection="handleDataSelection"
		></apexchart>
	</div>
</template>
