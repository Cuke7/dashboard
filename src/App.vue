<template>
    <div class="drawer">
        <input id="my-drawer" type="checkbox" class="drawer-toggle" />
        <div class="drawer-content flex flex-col w-full h-screen bg-slate-500">
            <!-- Page content here -->
            <!-- <label for="my-drawer" class="btn btn-primary drawer-button">Open drawer</label> -->
            <div class="flex bg-slate-800 p-4 justify-between items-center">
                <label class="flex cursor-pointer" for="my-drawer"> <Bars3Icon class="h-8 w-auto text-blue-500" /> </label>
                <span class="flex font-mono text-4xl text-white">⛵ LSK</span>
            </div>
            <div class="flex flex-wrap w-full p-2">
                <div class="flex flex-col w-full p-4 m-2 rounded-md bg-white space-y-2">
                    <div class="flex items-center space-x-4">
                        <label class="swap swap-rotate">
                            <input type="checkbox" v-model="play" />
                            <PlayIcon class="h-8 w-auto text-blue-500 swap-off" />
                            <PauseIcon class="h-8 w-auto text-blue-500 swap-on" />
                        </label>
                        <span class="text-black" v-if="!play"> Start plotting </span>
                        <span class="text-black" v-else> Pause plotting </span>
                    </div>
                    <div class="flex items-center space-x-4">
                        <input type="number" v-model="range" class="bg-transparent border-primary border-2 rounded-md p-1 w-16 text-black" />
                        <span class="text-black"> Chart plot window </span>
                    </div>
                    <div class="flex items-center justify-start space-x-4">
                        <input type="checkbox" class="toggle toggle-primary" v-model="animations" />
                        <span class="text-black"> Animations </span>
                    </div>
                </div>
                <div class="bg-white flex sm:p-4 p-2 rounded-lg shadow-2xl flex-grow sm:flex-1 m-2">
                    <apexchart class="w-full" type="line" :options="chartOption.line_chart.options" :series="chartData.line_chart.series"></apexchart>
                </div>
                <div class="bg-white flex sm:p-4 p-2 rounded-lg shadow-2xl flex-grow sm:flex-1 m-2">
                    <apexchart class="w-full" type="area" :options="chartOption.area_chart.options" :series="chartData.area_chart.series"></apexchart>
                </div>
                <div class="bg-white flex sm:p-4 p-2 rounded-lg shadow-2xl flex-grow sm:flex-1 m-2">
                    <apexchart class="w-full" type="radialBar" :options="chartOption.angle_chart.options" :series="chartData.angle_chart.series"></apexchart>
                </div>
                <div class="bg-white flex sm:p-4 p-2 rounded-lg shadow-2xl flex-grow sm:flex-1 m-2">
                    <apexchart class="w-full" type="bar" :options="chartOption.bar_chart.options" :series="chartData.bar_chart.series"></apexchart>
                </div>
            </div>
        </div>
        <div class="drawer-side">
            <label for="my-drawer" class="drawer-overlay"></label>
            <ul class="menu p-4 w-80 bg-base-100 text-base-content">
                <!-- Sidebar content here -->
                <li><a>Menu 1</a></li>
                <li><a>Menu 2</a></li>
            </ul>
        </div>
    </div>
</template>

<script lang="ts" setup>
import { ref, computed } from "vue";
import { Bars3Icon, PauseIcon, PlayIcon } from "@heroicons/vue/24/solid";

const range = ref(20);
const animations = ref(false);

const socket = new WebSocket("ws://localhost:1880/ws/simple");

socket.addEventListener("message", (event) => {
    if (!play.value) return;
    const data = JSON.parse(event.data);
    chartData.value.line_chart.series[0].data.push([data.time, data.data_line]);
    chartData.value.area_chart.series[0].data.push([data.time, data.data_area1]);
    chartData.value.area_chart.series[1].data.push([data.time, data.data_area2]);
    chartData.value.bar_chart.series[0].data[0] = data.data_area1;
    chartData.value.bar_chart.series[0].data[1] = data.data_area2;

    chartData.value.angle_chart.series = [Math.round((data.data_angle / 180) * 100)];

    for (const key of Object.keys(chartData.value)) {
        const chart = chartData.value[key];
        for (const serie of chart.series) {
            if (serie.data) {
                if (serie.data.length > range.value) {
                    serie.data.shift();
                }
            }
        }
    }
});

const play = ref(true);

const chartData = ref({
    line_chart: {
        series: [
            {
                name: "Line data",
                data: <any[]>[],
            },
        ],
    },
    area_chart: {
        series: [
            {
                name: "Capteur babord",
                data: <any[]>[],
            },
            {
                name: "Capteur tribord",
                data: <any[]>[],
            },
        ],
    },
    angle_chart: {
        series: [
            {
                name: "Angle data",
                series: <any>0,
            },
        ],
    },
    bar_chart: {
        series: [
            {
                data: [
                    {
                        x: "Une mesure",
                        y: 10,
                    },
                    {
                        x: "Une autre mesure",
                        y: 18,
                    },
                ],
            },
        ],
    },
});

const chartOption = computed(() => {
    return {
        line_chart: {
            options: {
                tooltip: {
                    enabled: false,
                },
                chart: {
                    animations: {
                        enabled: animations.value,
                    },
                },
                dataLabels: {
                    enabled: false,
                },
                xaxis: {
                    title: {
                        text: "Time (s)",
                    },
                    type: "datetime",
                },
                yaxis: {
                    title: {
                        text: "Distance (m)",
                    },
                },
                title: {
                    text: "Une mesure",
                },
            },
        },
        area_chart: {
            options: {
                tooltip: {
                    enabled: false,
                },
                chart: {
                    animations: {
                        enabled: animations.value,
                    },
                },
                dataLabels: {
                    enabled: false,
                },
                xaxis: {
                    title: {
                        text: "Time (s)",
                    },
                    type: "datetime",
                },
                yaxis: {
                    title: {
                        text: "Efforts (m)",
                    },
                },
                title: {
                    text: "Des capteurs",
                },
            },
        },
        angle_chart: {
            options: {
                title: {
                    text: "Un angle",
                },
                plotOptions: {
                    radialBar: {
                        startAngle: -90,
                        endAngle: 90,
                        track: {
                            background: "#333",
                            startAngle: -90,
                            endAngle: 90,
                        },
                        dataLabels: {
                            name: {
                                show: false,
                            },
                            value: {
                                fontSize: "30px",
                                show: true,
                                formatter: function (val: any) {
                                    return Math.round((val * 180) / 100) + "°";
                                },
                            },
                        },
                    },
                },
            },
        },
        bar_chart: {
            options: {
                title: {
                    text: "Des jauges",
                },
                yaxis: {
                    labels: {
                        show: false,
                    },
                    max: 600,
                },
            },
        },
    };
});
// Needs to be computed for animation switch to take effect
// const charts = ref(
// });

// const charts = ref({
//     line_data: [
//         {
//             name: "Line data",
//             data: <any>[],
//         },
//     ],
//     area_data: [
//         {
//             name: "Area data 1",
//             data: <any>[],
//         },
//         {
//             name: "Area data 2",
//             data: <any>[],
//         },
//     ],
// });
</script>
