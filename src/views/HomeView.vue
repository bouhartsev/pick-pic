<script lang="ts">
import MainCanvas from '../components/MainCanvas.vue';
import SideTools from '../components/SideTools.vue';
import StatusBar from '../components/StatusBar.vue';
import ColorPicker from '../components/ColorPicker.vue';
import ToolResize from '@/components/ToolResize.vue';
import ToolFiltration from '@/components/ToolFiltration.vue';
import ToolCurves from '@/components/ToolCurves.vue';

import { defineComponent } from 'vue';

interface HomeViewState {
  state: string;
  currentImage: HTMLImageElement | undefined;
  iw: number | undefined;
  ih: number | undefined;
  mouseCoordinates: [number, number];
  pickedColor: string;
  scale: number;
  isShiftPressed: boolean;

  interpolation: string;
  filterMatrix: number[][];
  colorData: {
    r: Array<number>;
    g: Array<number>;
    b: Array<number>;
  } | undefined;
  lut: Array<number> | undefined;
}

export default defineComponent({
  name: 'HomeView',
  components: {
    MainCanvas,
    SideTools,
    StatusBar,
    ColorPicker,
    ToolResize,
    ToolFiltration,
    ToolCurves,
  },

  data(): HomeViewState {
    return {
      state: '',
      currentImage: undefined,
      iw: undefined,
      ih: undefined,
      mouseCoordinates: [0, 0],
      pickedColor: "",
      scale: 100,
      isShiftPressed: false,

      interpolation: "default",
      filterMatrix: [
        [0, 0, 0],
        [0, 1, 0],
        [0, 0, 0],
      ],
      colorData: undefined,
      lut: [],
    };
  },

  mounted() {
    window.addEventListener('keydown', this.handlePressShiftKey);
    window.addEventListener('keyup', this.handleUnpressShiftKey);
  },
  beforeUnmount() {
    window.removeEventListener('keydown', this.handlePressShiftKey);
    window.removeEventListener('keyup', this.handleUnpressShiftKey);
  },
  watch: {
    state(newState: string) {
      if (newState === 'close') {
        this.currentImage = undefined;
        this.state = '';
      }
    },
  },
  methods: {
    handlePressShiftKey(event: KeyboardEvent): void {
      if (event.key === 'Shift') {
        this.isShiftPressed = true;
      }
    },
    handleUnpressShiftKey(event: KeyboardEvent): void {
      if (event.key === 'Shift') {
        this.isShiftPressed = false;
      }
    },

    changeState(newState: string): void {
      this.state = newState;
    },

    updateImage(image: HTMLImageElement): void {
      this.currentImage = image;
    },

    updateImageSizes(iw: number | undefined, ih: number | undefined): void {
      this.iw = iw;
      this.ih = ih;
    },

    updateCoordinates(coords: [number, number]): void {
      this.mouseCoordinates = coords;
    },

    updateActualSizes(w: number, h: number): void {
      if (!this.currentImage) return;
      const image = new Image();
      image.src = this.currentImage.src;
      image.width = w;
      image.height = h;
      this.currentImage = image;
    },
    updateWidth(w: number): void {
      if (!this.currentImage) return;
      this.updateActualSizes(w, this.currentImage.height);
    },
    updateHeight(h: number): void {
      if (!this.currentImage) return;
      this.updateActualSizes(this.currentImage.width, h);
    },

    updateFilterMatrix(matrix: number[][]): void {
      this.filterMatrix = matrix;
    },
  },

  computed: {
    hasImg(): boolean {
      return !!this.currentImage?.src;
    }
  }
});
</script>

<template>
  <main>
    <MainCanvas :state="state" :currentImg="currentImage" :iw="iw" :ih="ih" :pickedColor="pickedColor" :scale="scale"
      :isShiftPressed="isShiftPressed" :interpolation="interpolation" :filterMatrix="filterMatrix" :lut="lut"
      @updateImageSizes="updateImageSizes" @updateColor="(color) => (pickedColor = color)"
      @updateCoordinates="updateCoordinates" @changeState="changeState" @updateImg="updateImage"
      @updateColorData="(cData) => colorData = cData" />
  </main>
  <aside>
    <SideTools :state="state" :hasImg="hasImg" :scale="scale" @changeState="changeState"
      @updateScale="(value) => (scale = value)" />
    <div>
      <ColorPicker v-if="state === 'pipette'" :pickedColor="pickedColor" :mouseCoordinates="mouseCoordinates"
        :isShiftPressed="isShiftPressed" />
      <ToolResize v-if="state === 'edit'" :interpolation="interpolation"
        @updateInterpolation="(value) => interpolation = value" :width="currentImage?.width" @updateWidth="updateWidth"
        :height="currentImage?.height" @updateHeight="updateHeight" :ogWidth="currentImage?.naturalWidth"
        :ogHeight="currentImage?.naturalHeight" />
      <ToolCurves v-if="state === 'edit'" :colorData="colorData" @updateLUT="(val) => lut = val" />
      <ToolFiltration v-if="state === 'edit'" @updateFilterMatrix="updateFilterMatrix" />
    </div>
    <StatusBar v-if="hasImg" :state="state" :ih="currentImage?.height" :iw="currentImage?.width" :scale="scale" />
    <!-- :pickedColor="pickedColor" :xMouse="mouseCoordinates[0]" :yMouse="mouseCoordinates[1]" -->
  </aside>

</template>

<style scoped>
aside,
main {
  --size-height: calc(100vh - 2rem * 2 - 10vh);

  margin-top: 1em;
}

aside {
  min-height: var(--size-height);
  height: fit-content;
  width: 25%;


  display: flex;
  flex-direction: column;
  gap: 1rem;
  padding: 1rem;

  justify-content: space-between;
}

main {
  height: var(--size-height);
  width: 70%;
  margin-right: 5%;
  float: left;
  position: sticky;
  top: 2rem;
}
</style>
