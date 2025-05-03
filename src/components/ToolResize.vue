<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
    name: 'ResizeImage',
    props: {
        width: Number,
        height: Number,
        ogWidth: Number,
        ogHeight: Number,
        interpolation: String,
    },
    emits: ['updateWidth', 'updateHeight', 'updateInterpolation'],
    data() {
        return {
            resizeUnit: "pixels",
            whTogether: true
        };
    },
    computed: {
        newWidth: {
            get(): number {
                if (this.resizeUnit === 'pixels') return this.width ?? 0;
                return Math.round((this.width ?? 0) / (this.ogWidth ?? 1) * 100);
            },
            set(value: number) {
                if (Number.isNaN(value) || value < 1) return;

                if (this.whTogether) {
                    const coef = this.newWidth / this.newHeight;
                    this.whTogether = false;
                    this.newHeight = ~~(value * coef);
                    this.whTogether = true;
                }
                if (this.resizeUnit === 'pixels') {
                    this.$emit('updateWidth', value);
                } else {
                    this.$emit('updateWidth', (this.ogWidth ?? 0) * value / 100);
                }
            },
        },
        newHeight: {
            get(): number {
                if (this.resizeUnit === 'pixels') return this.height ?? 0;
                return Math.round((this.height ?? 0) / (this.ogHeight ?? 1) * 100);
            },
            set(value: number) {
                if (Number.isNaN(value) || value < 1) return;

                if (this.whTogether) {
                    const coef = this.newWidth / this.newHeight;
                    this.whTogether = false;
                    this.newWidth = ~~(value * coef);
                    this.whTogether = true;
                }
                if (this.resizeUnit === 'pixels') {
                    this.$emit('updateHeight', value);
                } else {
                    this.$emit('updateHeight', (this.ogHeight ?? 0) * value / 100);
                }
            },
        },
        newInterpolation: {
            get(): string { return this.interpolation ?? 'default' },
            set(value: string) { this.$emit('updateInterpolation', value) },
        },
    }
});
</script>

<template>
    <div class="container">
        <div class="resize">
            <div class="units">
                <label>
                    <input type="radio" name="resizeUnit" value="pixels" checked v-model="resizeUnit">
                    <span>Pixels</span>
                </label>
                <label>
                    <input type="radio" name="resizeUnit" value="percentage" v-model="resizeUnit">
                    <span>Percentage</span>
                </label>
            </div>
            <div class="sizes">
                <div class="input-with-suffix">
                    <input id="width" v-model="newWidth" type="number" />
                    <label for="width" class="type">
                        {{ resizeUnit === "pixels" ? "px" : "%" }}
                    </label>
                </div>
                <div class="custom-checkbox">
                    <input type="checkbox" v-model="whTogether" id="lock" />
                    <label for="lock" class="lock-label" title="Change width and height proportionnally">
                        <span class="lock-wrapper">
                            <span class="shackle"></span>
                            <svg class="lock-body" width="" height="" viewBox="0 0 28 28" fill="none"
                                xmlns="http://www.w3.org/2000/svg">
                                <path fill-rule="evenodd" clip-rule="evenodd"
                                    d="M0 5C0 2.23858 2.23858 0 5 0H23C25.7614 0 28 2.23858 28 5V23C28 25.7614 25.7614 28 23 28H5C2.23858 28 0 25.7614 0 23V5ZM16 13.2361C16.6137 12.6868 17 11.8885 17 11C17 9.34315 15.6569 8 14 8C12.3431 8 11 9.34315 11 11C11 11.8885 11.3863 12.6868 12 13.2361V18C12 19.1046 12.8954 20 14 20C15.1046 20 16 19.1046 16 18V13.2361Z"
                                    fill="white"></path>
                            </svg>
                        </span>
                    </label>
                </div>
                <div class="input-with-suffix">
                    <input id="height" v-model="newHeight" type="number" />
                    <label for="height" class="type">
                        {{ resizeUnit === "pixels" ? "px" : "%" }}
                    </label>
                </div>
            </div>
            <div class="interpolation">
                <label for="interpolation">Interpolation:&nbsp;</label>
                <select v-model="newInterpolation" id="interpolation">
                    <option value="default" selected>default</option>
                    <option value="nearestNeighbor"
                        title="Each pixel in the new image is assigned the value of the nearest pixel in the original image">
                        Nearest Neighbor
                    </option>
                </select>
            </div>
        </div>
        <hr />
        <!--  -->

    </div>
</template>

<style scoped>
.sizes {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 1rem;

    margin-block: 1rem;
}

.input-with-suffix>input {
    width: 5rem;
}


/* From Uiverse.io by vinodjangid07 */
#lock {
    display: none;
}

.lock-label {
    width: 45px;
    height: 45px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: rgb(80, 80, 80);
    border-radius: 15px;
    cursor: pointer;
    transition: all 0.3s;
}

.lock-wrapper {
    width: fit-content;
    height: fit-content;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    transform: rotate(-10deg);
}

.shackle {
    background-color: transparent;
    height: 9px;
    width: 14px;
    border-top-right-radius: 10px;
    border-top-left-radius: 10px;
    border-top: 3px solid white;
    border-left: 3px solid white;
    border-right: 3px solid white;
    transition: all 0.3s;
}

.lock-body {
    width: 15px;
}

#lock:not(:checked)+.lock-label .lock-wrapper .shackle {
    transform: rotateY(150deg) translateX(3px);
    transform-origin: right;
}

#lock:not(:checked)+.lock-label {
    background-color: rgb(167, 71, 245);
}

.lock-label:active {
    transform: scale(0.9);
}

/* From Uiverse.io by Pradeepsaranbishnoi */
.units :focus {
    outline: 0;
    border-color: #2260ff;
    box-shadow: 0 0 0 4px #b5c9fc;
}

.units {
    display: flex;
    flex-wrap: wrap;
    margin-top: 0.5rem;
    justify-content: center;
}

.units input[type="radio"] {
    clip: rect(0 0 0 0);
    clip-path: inset(100%);
    height: 1px;
    overflow: hidden;
    position: absolute;
    white-space: nowrap;
    width: 1px;
}

.units input[type="radio"]:checked+span {
    box-shadow: 0 0 0 0.0625em var(--color-blue);
    background-color: var(--vt-c-black-mute);
    z-index: 1;
    color: var(--color-blue);
}

.units label span {
    display: block;
    cursor: pointer;
    /* background-color: #fff; */
    padding: 0.375em .75em;
    position: relative;
    margin-left: .0625em;
    box-shadow: 0 0 0 0.0625em #b5bfd9;
    letter-spacing: .05em;
    /* color: #3e4963; */
    text-align: center;
    transition: background-color .5s ease;
}

.units label:first-child span {
    border-radius: .375em 0 0 .375em;
}

.units label:last-child span {
    border-radius: 0 .375em .375em 0;
}

.interpolation {
    text-align: center;
}
</style>
