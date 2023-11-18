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

const props = defineProps([
	"chart_config",
	"activeChart",
	"series",
	"map_config",
]);

const chartRef = ref(null);

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
		},
	},
	legend: {
		show: false,
	},
	grid: {
		show: false,
	},
	markers: {
		hover: {
			size: 8,
		},
	},
	tooltip: {
		custom: function ({ series, seriesIndex, dataPointIndex, w }) {
			return (
				'<div class="chart-tooltip">' +
				"<h6>" +
				props.series[seriesIndex].name +
				"</h6>" +
				"<span>" +
				`${props.chart_config.xaxisUnit}:${w.globals.seriesX[seriesIndex][dataPointIndex]}` +
				"<br />" +
				`${props.chart_config.yaxisUnit}:${series[seriesIndex][dataPointIndex]}` +
				"</span>" +
				"</div>"
			);
		},
		followCursor: true,
	},
	xaxis: {
		tickAmount: 5,
		axisTicks: {
			show: false,
		},
		tooltip: {
			enabled: false,
		},
	},
	yaxis: {
		tickAmount: 5,
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
	if (props.chart_config.backgrounds.length === 0) {
		return;
	}
	const bgs = props.chart_config.backgrounds;
	const { width, height } = getSize();
	const ns = "http://www.w3.org/2000/svg";
	const foreignObject = document.createElementNS(ns, "foreignObject");

	foreignObject.setAttribute("x", 0);
	foreignObject.setAttribute("y", 0);
	foreignObject.setAttribute("width", width);
	foreignObject.setAttribute("height", height);

	for (let i = 0; i < bgs.length; i++) {
		const bg = bgs[i];
		const div = document.createElement("div");
		div.setAttribute(
			"style",
			`position: absolute; width: ${bg.width}; height: ${
				bg.height
			}; top: ${bg.top}; left: ${bg.left}; border: 2px solid ${
				bg.color
			}; box-sizing: border-box; background: ${hexToRgba(bg.color, 0.2)}`
		);
		foreignObject.appendChild(div);
	}
	chartRef.value.$el.querySelector(".apexcharts-grid").append(foreignObject);
};

const getSize = () => {
	const element = chartRef.value.$el.querySelector(".apexcharts-grid");
	const domData = element.getBoundingClientRect();

	return {
		width: domData.width,
		height: domData.height,
	};
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
				v-for="backgroundSetting in props.chart_config.backgrounds"
				:key="backgroundSetting.color"
				class="apexcharts-legend-series"
				:style="{
					display: 'flex',
					alignItems: 'center',
					justifyContent: 'center',
					marginLeft: '10px',
				}"
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
