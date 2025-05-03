<script lang="ts">
import { defineComponent } from "vue";
import Chart from "chart.js/auto";

export default defineComponent({
    name: "ToolCurves",
    props: {
        colorData: Object, // { r: Array<number>, g: Array<number>, b: Array<number> }

    },
    emits: ["updateLUT"],
    mounted() {
        this.chartRef = this.$refs.chart as HTMLCanvasElement;
        this.renderGraph();
    },
    beforeUnmount() {
        if (this.chartInstance) {
            this.chartInstance.destroy();
            this.chartInstance = null;
        }
    },
    data(): {
        point1: [number, number];
        point2: [number, number];
        liveUpdate: boolean;
        // chart.js
        chartInstance: Chart | null;
        chartRef: HTMLCanvasElement | null;
    } {
        return {
            point1: [0, 0],
            point2: [255, 255],
            liveUpdate: true,

            chartInstance: null,
            chartRef: null,
        };
    },
    methods: {
        renderGraph() {
            const ctx = this.chartRef?.getContext("2d");
            if (!ctx) return;

            const { r: rData, g: gData, b: bData } = this.colorData ?? {};

            if (this.chartInstance) {
                this.chartInstance.destroy();
                this.chartInstance = null;
            }

            // TODO: fix TS error and actual error with rerendering
            this.chartInstance = new Chart(ctx, {
                type: "line",
                data: {
                    labels: Array.from({ length: 256 }, (_, i) => i),
                    datasets: [
                        {
                            label: "Red",
                            data: rData,
                            backgroundColor: "rgba(255, 0, 0, 0.8)",
                            borderWidth: 0.5,
                            borderColor: "rgba(255, 255, 255, 1)",
                        },
                        {
                            label: "Green",
                            data: gData,
                            backgroundColor: "rgba(0, 255, 0, 0.8)",
                            borderWidth: 0.5,
                            borderColor: "rgba(255, 255, 255, 1)",
                        },
                        {
                            label: "Blue",
                            data: bData,
                            backgroundColor: "rgba(0, 0, 255, 0.8)",
                            borderWidth: 0.5,
                            borderColor: "rgba(255, 255, 255, 1)",
                        },
                        {
                            label: "Correction",
                            data: [
                                { x: 0, y: this.point1[1] },
                                { x: this.point1[0], y: this.point1[1] },
                                { x: this.point2[0], y: this.point2[1] },
                                { x: 255, y: this.point2[1] },
                            ],
                            borderColor: "rgba(255, 255, 255, 1)",
                            borderWidth: 1,
                            fill: false,
                        },
                    ],
                },
                options: {
                    animation: {
                        duration: 500,
                    },
                    scales: {
                        x: {
                            ticks: {
                                color: 'white'
                            },
                            grid: {
                                color: 'rgba(255, 255, 255, 0.1)'
                            }
                        },
                        y: {
                            beginAtZero: true,
                            ticks: {
                                color: 'white'
                            },
                            grid: {
                                color: 'rgba(255, 255, 255, 0.1)'
                            }
                        },
                    },
                    plugins: {
                        legend: {
                            labels: {
                                color: 'white'
                            }
                        }
                    }
                },
            });
        },
        adjust() {
            const lut = []; // look-up table
            for (let i = 0; i < this.point1[0]; i++) {
                lut[i] = this.point1[1];
            }
            for (let i = this.point1[0]; i < this.point2[0]; i++) {
                const slope = (this.point2[1] - this.point1[1]) / (this.point2[0] - this.point1[0]);
                let val = this.point1[1] + slope * (i - this.point1[0]);
                val = Math.max(0, Math.min(255, val));
                lut[i] = val;
            }
            for (let i = this.point2[0]; i < 256; i++) {
                lut[i] = this.point2[1];
            }

            this.$emit("updateLUT", lut);
        },
        apply() {
            this.adjust();
            // TODO: update img
        },
        resetAll() {
            this.point1 = [0, 0];
            this.point2 = [255, 255];
            this.$emit("updateLUT", []);
            // TODO: reset img

        },
        updateX1(event: Event) {
            const target = event.target as HTMLInputElement;
            const num = +target.value;

            if (!isNaN(num) && num >= 0 && num < this.point2[0]) {
                this.point1[0] = num;
                this.renderGraph();
                if (this.liveUpdate) {
                    this.adjust();
                }
            } else {
                this.point1[0] += 1;
                this.point1[0] -= 1;
            }
        },
        updateY1(event: Event) {
            const target = event.target as HTMLInputElement;
            const num = +target.value;

            if (!isNaN(num) && num >= 0 && num < 255) {
                this.point1[1] = num;
                this.renderGraph();
                if (this.liveUpdate) {
                    this.adjust();
                }
            } else {
                this.point1[1] += 1;
                this.point1[1] -= 1;
            }
        },
        updateX2(event: Event) {
            const target = event.target as HTMLInputElement;
            const num = +target.value;

            if (!isNaN(num) && num <= 255 && this.point1[0] <= num) {
                this.point2[0] = num;
                this.renderGraph();
                if (this.liveUpdate) {
                    this.adjust();
                }
            } else {
                this.point2[0] += 1;
                this.point2[0] -= 1;
            }
        },
        updateY2(event: Event) {
            const target = event.target as HTMLInputElement;
            const num = +target.value;

            if (!isNaN(num) && num <= 255 && 0 <= num) {
                this.point2[1] = num;
                this.renderGraph();
                if (this.liveUpdate) {
                    this.adjust();
                }
            } else {
                this.point2[1] += 1;
                this.point2[1] -= 1;
            }
        },
    },
    watch: {
        // colorData: {
        //     handler() {
        //         this.renderGraph();
        //     },
        //     // deep: true
        // },
    },
});
</script>

<template>
    <div class="curve-tool">
        <div class="curve-inputs">
            <div class="input-group">
                <div class="labeled-input">
                    <label for="x1">x1
                        <input id="x1" :value="point1[0]" @change="updateX1" />
                    </label>
                </div>
                <div class="labeled-input">
                    <label for="y1">y1
                        <input id="y1" :value="point1[1]" @change="updateY1" />
                    </label>
                </div>
            </div>
            <div class="input-group">
                <div class="labeled-input">
                    <label for="x2">x2
                        <input id="x2" :value="point2[0]" @change="updateX2" />
                    </label>
                </div>
                <div class="labeled-input">
                    <label for="y2">y2
                        <input id="y2" :value="point2[1]" @change="updateY2" />
                    </label>
                </div>
            </div>
        </div>
        <div class="curve-graph">
            <canvas id="chart" width="100" height="100" ref="chart"></canvas>
        </div>
        <div class="curve-actions">
            <label class="toggle-preview" title="Show preview on change">
                <input type="checkbox" v-model="liveUpdate">
                <svg class="eye" xmlns="http://www.w3.org/2000/svg" height="1em" viewBox="0 0 576 512">
                    <path
                        d="M288 32c-80.8 0-145.5 36.8-192.6 80.6C48.6 156 17.3 208 2.5 243.7c-3.3 7.9-3.3 16.7 0 24.6C17.3 304 48.6 356 95.4 399.4C142.5 443.2 207.2 480 288 480s145.5-36.8 192.6-80.6c46.8-43.5 78.1-95.4 93-131.1c3.3-7.9 3.3-16.7 0-24.6c-14.9-35.7-46.2-87.7-93-131.1C433.5 68.8 368.8 32 288 32zM144 256a144 144 0 1 1 288 0 144 144 0 1 1 -288 0zm144-64c0 35.3-28.7 64-64 64c-7.1 0-13.9-1.2-20.3-3.3c-5.5-1.8-11.9 1.6-11.7 7.4c.3 6.9 1.3 13.8 3.2 20.7c13.7 51.2 66.4 81.6 117.6 67.9s81.6-66.4 67.9-117.6c-11.1-41.5-47.8-69.4-88.6-71.1c-5.8-.2-9.2 6.1-7.4 11.7c2.1 6.4 3.3 13.2 3.3 20.3z">
                    </path>
                </svg>
                <svg class="eye-slash" xmlns="http://www.w3.org/2000/svg" height="1em" viewBox="0 0 640 512">
                    <path
                        d="M38.8 5.1C28.4-3.1 13.3-1.2 5.1 9.2S-1.2 34.7 9.2 42.9l592 464c10.4 8.2 25.5 6.3 33.7-4.1s6.3-25.5-4.1-33.7L525.6 386.7c39.6-40.6 66.4-86.1 79.9-118.4c3.3-7.9 3.3-16.7 0-24.6c-14.9-35.7-46.2-87.7-93-131.1C465.5 68.8 400.8 32 320 32c-68.2 0-125 26.3-169.3 60.8L38.8 5.1zM223.1 149.5C248.6 126.2 282.7 112 320 112c79.5 0 144 64.5 144 144c0 24.9-6.3 48.3-17.4 68.7L408 294.5c8.4-19.3 10.6-41.4 4.8-63.3c-11.1-41.5-47.8-69.4-88.6-71.1c-5.8-.2-9.2 6.1-7.4 11.7c2.1 6.4 3.3 13.2 3.3 20.3c0 10.2-2.4 19.8-6.6 28.3l-90.3-70.8zM373 389.9c-16.4 6.5-34.3 10.1-53 10.1c-79.5 0-144-64.5-144-144c0-6.9 .5-13.6 1.4-20.2L83.1 161.5C60.3 191.2 44 220.8 34.5 243.7c-3.3 7.9-3.3 16.7 0 24.6c14.9 35.7 46.2 87.7 93 131.1C174.5 443.2 239.2 480 320 480c47.8 0 89.9-12.9 126.2-32.5L373 389.9z">
                    </path>
                </svg>
            </label>
            <button class="button action-button" @click="apply">
                Apply
            </button>
            <button class="button action-button" @click="resetAll">Reset</button>
        </div>
        <hr />
    </div>
</template>

<style scoped>
.curve-inputs {
    display: flex;
    flex-direction: column;
    gap: 5px;
    align-items: center;
}

.input-group {
    display: flex;
    gap: 2rem;
}

.labeled-input {
    position: relative;
}

.labeled-input input {
    height: 40px;
    width: 64px;
    border: 1px solid #ccc;
    border-radius: 4px;
    padding-left: 4px;
    text-align: center;
}

.preview-control {
    position: relative;
    display: inline-block;
}

.custom-checkbox input,
.custom-checkbox .checkbox-image {
    width: 20px;
    height: 20px;
    cursor: pointer;
}

.custom-checkbox input[type="checkbox"] {
    position: absolute;
    opacity: 0;
    cursor: pointer;
}

.custom-checkbox input[type="checkbox"]:hover {
    border-color: #a0a0a0;
}

.curve-actions {
    display: flex;
    justify-content: center;
    gap: 0.5em;
    margin-top: 0.5em;
}

.curve-graph {
    font-size: small;
    display: flex;
    flex-direction: column;
    align-items: center;
}


/* From Uiverse.io by catraco */
/*------ Settings ------*/
.toggle-preview {
    --color: currentColor;
    --size: 30px;
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
    cursor: pointer;
    font-size: var(--size);
    user-select: none;
    fill: var(--color);

    width: 2rem;
    height: 2rem;
}

.toggle-preview .eye {
    position: absolute;
    animation: keyframes-fill .5s;
}

.toggle-preview .eye-slash {
    position: absolute;
    animation: keyframes-fill .5s;
    display: none;
}

/* ------ On check event ------ */
.toggle-preview input:checked~.eye {
    display: none;
}

.toggle-preview input:checked~.eye-slash {
    display: block;
}

/* ------ Hide the default checkbox ------ */
.toggle-preview input {
    position: absolute;
    opacity: 0;
    cursor: pointer;
    height: 0;
    width: 0;
}

/* ------ Animation ------ */
@keyframes keyframes-fill {
    0% {
        transform: scale(0);
        opacity: 0;
    }

    50% {
        transform: scale(1.2);
    }
}
</style>
