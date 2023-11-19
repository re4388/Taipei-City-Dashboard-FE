<!-- Developed by Taipei Urban Intelligence Center 2023 -->

<script setup>
import { ref, computed, onMounted, nextTick } from "vue";
import { useMapStore } from "../../store/mapStore";

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
const mapStore = useMapStore();

const chartRef = ref(null);

const lastClickedQuadrant = ref(null);

const backgroundSettings = computed(() => {
	return props.chart_config.backgrounds.map((item, index) => {
		return Object.assign(DEFAULT_BACKGROUND_CONFIG[index], item);
	});
});

const legendClickHandler = (quadrant) => {
	chartRef.value.resetSeries();

	if (props.chart_config.map_filter) {
		props.map_config.map((item) => {
			mapStore.clearLayerFilter(`${item.index}-${item.type}`);
		});
	}

	if (lastClickedQuadrant.value === quadrant) {
		lastClickedQuadrant.value = null;
		return;
	}

	switch (quadrant) {
		case 0:
			props.series.map(({ data, name }) => {
				if (!(data.x < X_AXIS_MID && data.y > Y_AXIS_MID))
					chartRef.value.toggleSeries(name);
			});
			lastClickedQuadrant.value = 0;
			break;
		case 1:
			props.series.map(({ data, name }) => {
				if (!(data.x > X_AXIS_MID && data.y > Y_AXIS_MID))
					chartRef.value.toggleSeries(name);
			});
			lastClickedQuadrant.value = 1;
			break;
		case 2:
			props.series.map(({ data, name }) => {
				if (!(data.x < X_AXIS_MID && data.y < Y_AXIS_MID))
					chartRef.value.toggleSeries(name);
			});
			lastClickedQuadrant.value = 2;
			break;
		case 3:
			props.series.map(({ data, name }) => {
				if (!(data.x > X_AXIS_MID && data.y < Y_AXIS_MID))
					chartRef.value.toggleSeries(name);
			});
			lastClickedQuadrant.value = 3;
			break;

		default:
			lastClickedQuadrant.value = null;
			break;
	}

	if (!props.chart_config.map_filter) {
		return;
	}

	props.map_config.map((item) => {
		mapStore.addLayerFilter(
			`${item.index}-${item.type}`,
			props.chart_config.map_filter[0],
			props.chart_config.map_filter[1][quadrant]
		);
	});
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
	const apexchartsCanvas =
		chartRef.value.$el.querySelector(".apexcharts-canvas");

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

	for (const [key, value] of Object.entries(
		props.chart_config.labels || []
	)) {
		const div = document.createElement("div");
		div.textContent = value;

		switch (key) {
			case "x1":
				div.setAttribute(
					"style",
					`
                    position: absolute;
                    bottom: 0%;
                    left: 3%;
                    font-size: 12px;
                    `
				);
				break;
			case "x2":
				div.setAttribute(
					"style",
					`
                    position: absolute;
                    bottom: 0;
                    left: 50%;
                    font-size: 12px;
                    `
				);
				break;
			case "y1":
				div.setAttribute(
					"style",
					`
                    position: absolute;
                    bottom: 8%;
                    left: -1%;
                    writing-mode:vertical-lr;
                    font-size: 12px;
                    `
				);
				break;
			case "y2":
				div.setAttribute(
					"style",
					`
                    position: absolute;
                    bottom: 50%;
                    left: -1%;
                    writing-mode:vertical-lr;
                    font-size: 12px;
                    `
				);
				break;
			default:
				break;
		}

		apexchartsCanvas.appendChild(div);
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
					opacity:
						lastClickedQuadrant === null
							? 1
							: lastClickedQuadrant === index
							? 1
							: 0.5,
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
