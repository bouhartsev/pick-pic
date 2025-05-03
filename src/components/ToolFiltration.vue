<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
    name: "ToolFiltration",
    emits: [
        'updateFilterMatrix'
    ],
    data() {
        return {
            liveUpdate: false,
            filterMatrix: [
                [0, 0, 0],
                [0, 1, 0],
                [0, 0, 0],
            ],
            filterProfile: "identity",
        };
    },
    methods: {
        setFilterProfile(profile: string) {
            switch (profile) {
                case "identity":
                    this.filterMatrix = [
                        [0, 0, 0],
                        [0, 1, 0],
                        [0, 0, 0],
                    ];
                    break;
                case "sharpen":
                    this.filterMatrix = [
                        [0, -1, 0],
                        [-1, 5, -1],
                        [0, -1, 0],
                    ];
                    break;
                case "gaussian":
                    this.filterMatrix = [
                        [1, 2, 1],
                        [2, 4, 2],
                        [1, 2, 1],
                    ];
                    break;
                case "boxBlur":
                    this.filterMatrix = [
                        [1, 1, 1],
                        [1, 1, 1],
                        [1, 1, 1],
                    ];
                    break;
                default:
                    break;
            }
        },
        adjustMatrixCell(event: Event, rowIndex: number, colIndex: number) {
            const target = event.target as HTMLInputElement;
            const num = +target.value;

            if (!isNaN(num)) {
                this.filterMatrix[rowIndex][colIndex] = num;
            } else {
                this.filterMatrix[rowIndex][colIndex] += 1;
                this.filterMatrix[rowIndex][colIndex] -= 1;
            }
        },

        applyChanges() {
            // TODO: fix APPLY and PREVIEW - preview doesn't really resets and apply doesn't really applies
            this.$emit('updateFilterMatrix', this.filterMatrix);
        },
        clearFilter() {
            this.liveUpdate = false;
            this.filterProfile = "identity";
        },
    },
    watch: {
        filterMatrix: {
            handler() {
                if (this.liveUpdate) {
                    this.applyChanges();
                }
            },
            deep: true
        },
        filterProfile(newValue: string) {
            this.setFilterProfile(newValue);
        },

    },
});
</script>

<template>
    <div class="matrix-controls">
        <select class="matrix-preset" v-model="filterProfile">
            <option value="identity" title="No changes to the image">
                Identity Transformation
            </option>
            <option value="sharpen" title="Enhances details">Sharpen</option>
            <option value="gaussian" title="Smooth blur with natural falloff">
                Gaussian Filter
            </option>
            <option value="boxBlur" title="Uniform blur effect">Box Blur</option>
        </select>
        <div class="matrix-content">
            <div class="matrix-inputs">
                <div v-for="(row, rowIndex) in filterMatrix" class="matrix-row" :key="rowIndex">
                    <input v-for="(value, colIndex) in row" :value="filterMatrix[rowIndex][colIndex]"
                        @change="(e: Event) => adjustMatrixCell(e, rowIndex, colIndex)" :key="colIndex" />
                </div>
            </div>
            <div class="matrix-actions">
                <!-- liveUpdate control -->
                <button class="button matrix-button" @click="applyChanges">
                    Apply
                </button>
                <button class="button matrix-button" @click="clearFilter">
                    Clear
                </button>
            </div>
        </div>
        <hr />
    </div>
</template>

<style>
.matrix-preset {
    width: 100%;
    padding: 4px;
    border: 1px solid #ccc;
    border-radius: 4px;
    margin-bottom: 1rem;
}

.matrix-content {
    display: flex;
    flex-direction: column;
    gap: 1em;
}

.matrix-inputs {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.matrix-row {
    display: flex;
    /* flex-direction: column; */
    gap: 4px;
}

.matrix-row input {
    height: 40px;
    width: 48px;
    border: 1px solid #ccc;
    border-radius: 4px;
    padding-left: 4px;
    text-align: center;
}

.matrix-actions {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 1em;
}
</style>
