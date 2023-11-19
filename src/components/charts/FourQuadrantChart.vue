<!-- Developed by Taipei Urban Intelligence Center 2023 -->

<script setup>
import { ref, computed, onMounted, nextTick } from "vue";

const hexToRgba = (hex, opacity) => {
	const hexColor = hex.replace("#", "");
	const r = parseInt(hexColor.substring(0, 2), 16);
	const g = parseInt(hexColor.substring(2, 4), 16);
	const b = parseInt(hexColor.substring(4, 6), 16);
	return `rgba(${r}, ${g}, ${b}, ${opacity})`;
};

const X_AXIS_MIN = -1;
const X_AXIS_MAX = 8;
const Y_AXIS_MIN = -1;
const Y_AXIS_MAX = 8;
const X_AXIS_MID = 3.5;
const Y_AXIS_MID = 3.5;

const DEFAULT_BACKGROUND_CONFIG = [
	{
		color: "#f75d52",
		top: "0",
		left: "0",
	},
	{
		color: "#29c22b",
		top: "0",
		left: "50%",
	},
	{
		color: "#f5c489",
		top: "50%",
		left: "0",
	},
	{
		color: "#436ce8",
		top: "50%",
		left: "50%",
	},
];

const props = defineProps([
	"chart_config",
	"activeChart",
	"series",
	"map_config",
]);

const chartRef = ref(null);

const backgroundSettings = computed(() => {
	return props.chart_config.backgrounds.map((item, index) => {
		return Object.assign(DEFAULT_BACKGROUND_CONFIG[index], item);
	});
});

const legendClickHandler = (quadrant) => {
	switch (quadrant) {
		case 0:
			props.series.map(({ data, name }) => {
				if (data.x < X_AXIS_MID && data.y > Y_AXIS_MID)
					chartRef.value.toggleSeries(name);
			});
			break;
		case 1:
			props.series.map(({ data, name }) => {
				if (data.x > X_AXIS_MID && data.y > Y_AXIS_MID)
					chartRef.value.toggleSeries(name);
			});
			break;
		case 2:
			props.series.map(({ data, name }) => {
				if (data.x < X_AXIS_MID && data.y < Y_AXIS_MID)
					chartRef.value.toggleSeries(name);
			});
			break;
		case 3:
			props.series.map(({ data, name }) => {
				if (data.x > X_AXIS_MID && data.y < Y_AXIS_MID)
					chartRef.value.toggleSeries(name);
			});
			break;

		default:
			break;
	}
};

const chartOptions = ref({
	chart: {
		type: "scatter",
		toolbar: {
			show: false,
		},
		zoom: {
			enabled: false,
		},
	},
	dataLabels: {
		enabled: true,
		formatter: function (val, opts) {
			const { name } = opts.w.config.series[opts.seriesIndex];
			return name;
		},
		textAnchor: "middle",
		offsetX: 10,
		offsetY: 4,
		style: {
			fontSize: "10px",
			colors: ["#1f1d1d"],
		},
		background: {},
	},
	colors: ["#1f1d1d"],
	legend: {
		show: false,
	},
	grid: {
		show: false,
	},
	markers: {
		colors: ["#1f1d1d"],
		hover: {
			size: 8,
		},
	},
	tooltip: {
		custom: function ({ seriesIndex }) {
			return (
				'<div class="chart-tooltip">' +
				"<h6>" +
				props.series[seriesIndex].name +
				"</h6>" +
				"<span>" +
				`${
					props.chart_config.xaxisLabel
				}:${new Intl.NumberFormat().format(
					props.series[seriesIndex].data.x_value
				)}${props.chart_config.xaxisUnit}` +
				"<br />" +
				`${
					props.chart_config.yaxisLabel
				}:${new Intl.NumberFormat().format(
					props.series[seriesIndex].data.y_value
				)}${props.chart_config.yaxisUnit}` +
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
		show: false,
		tickAmount: 5,
		min: X_AXIS_MIN,
		max: X_AXIS_MAX,
		labels: {
			show: false,
		},
		axisTicks: {
			show: false,
		},
		tooltip: {
			enabled: false,
		},
	},
	yaxis: {
		show: false,
		tickAmount: 5,
		min: Y_AXIS_MIN,
		max: Y_AXIS_MAX,
	},
	stroke: {
		colors: ["#282a2c"],
		show: true,
		width: 0,
	},
});

onMounted(() => {
	nextTick(() => {
		drawBg();
	});
});

const drawBg = () => {
	if (backgroundSettings.value.length === 0) return;

	const ele = chartRef.value.$el.querySelector(".apexcharts-grid");

	if (ele.querySelector("foreignObject")) return;

	const bgs = backgroundSettings.value;
	const { width, height } = ele.getBoundingClientRect();
	const ns = "http://www.w3.org/2000/svg";
	const foreignObject = document.createElementNS(ns, "foreignObject");

	foreignObject.setAttribute("x", 0);
	foreignObject.setAttribute("y", 0);
	foreignObject.setAttribute("width", width);
	foreignObject.setAttribute("height", height);

	for (let i = 0; i < bgs.length; i++) {
		const bg = bgs[i];
		const div = document.createElement("div");

		const color = bg.color ? bg.color : DEFAULT_BACKGROUND_CONFIG[i].color;

		const transparency = color === "#ffffff" ? 0 : 0.2;

		div.setAttribute(
			"style",
			`position: absolute;
			width: 50%;
			height: 50%;
			top: ${DEFAULT_BACKGROUND_CONFIG[i].top};
			left: ${DEFAULT_BACKGROUND_CONFIG[i].left};
			border: 2px solid ${hexToRgba(color, transparency)};
			box-sizing: border-box; background: ${hexToRgba(color, transparency)}`
		);
		foreignObject.appendChild(div);
	}

	ele.append(foreignObject);
};

const onChartUpdated = () => {
	drawBg();
};

const parsedSeries = computed(() => {
	return props.series.map(({ name, data }) => {
		return {
			name: name,
			data: [[data.x, data.y]],
		};
	});
});
</script>

<template>
	<div v-if="activeChart === 'FourQuadrantChart'" class="four-quadrant-chart">
		<apexchart
			ref="chartRef"
			width="100%"
			height="240px"
			type="scatter"
			:options="chartOptions"
			:series="parsedSeries"
			@updated="onChartUpdated"
		></apexchart>

		<div
			class="apexcharts-legend apexcharts-align-center apx-legend-position-bottom"
			:style="{
				display: 'flex',
				alignItems: 'center',
				justifyContent: 'center',
			}"
		>
			<div
				v-for="(backgroundSetting, index) in backgroundSettings"
				:key="backgroundSetting.color"
				class="apexcharts-legend-series"
				:style="{
					display: 'flex',
					alignItems: 'center',
					justifyContent: 'center',
					marginLeft: '10px',
				}"
				@click="legendClickHandler(index)"
			>
				<div
					class="apexcharts-legend-marker"
					:style="{
						marginRight: '4px',
						width: '12px',
						height: '12px',
						borderWidth: '0px',
						borderColor: 'rgb(255, 255, 255)',
						borderRadius: '12px',
						backgroundColor: backgroundSetting.color,
						color: backgroundSetting.color,
					}"
				></div>
				<span
					class="apexcharts-legend-text"
					:style="{
						color: 'rgb(55, 61, 63)',
						fontSize: '12px',
						fontWeight: 400,
						fontFamily: 'Helvetica, Arial, sans-serif',
					}"
				>
					{{ backgroundSetting.name }}
				</span>
			</div>
		</div>
	</div>
</template>
