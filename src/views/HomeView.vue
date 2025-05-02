<script lang="ts">
import MainCanvas from '../components/MainCanvas.vue';
import SideTools from '../components/SideTools.vue';
import StatusBar from '../components/StatusBar.vue';
import ColorPicker from '../components/ColorPicker.vue';
import EditImage from '@/components/EditImage.vue';

export default {
  name: 'HomeView',
  components: {
    MainCanvas,
    SideTools,
    StatusBar,
    ColorPicker,
    EditImage
  },
  data() {
    return {
      state: '',

      currentImage: new Image(),

      iw: undefined,
      ih: undefined,
      mouseCoordinates: [0, 0],
      pickedColor: "",
      scale: 100,

      ogWidth: undefined,
      ogHeight: undefined,

      interpolation: "default",

      isShiftPressed: false,
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
  methods: {
    handlePressShiftKey(event) {
      if (event.key === 'Shift') {
        this.isShiftPressed = true;
      }
    },
    handleUnpressShiftKey(event) {
      if (event.key === 'Shift') {
        this.isShiftPressed = false;
      }
    },

    changeState(newState: string) {
      this.state = newState;
    },

    updateImage(image) {
      this.currentImage = image;

      this.ogWidth = image.width;
      this.ogHeight = image.height;
    },

    updateImageSizes(iw, ih) {
      this.iw = iw;
      this.ih = ih;
    },

    updateCoordinates(coords) { // [x, y]
      this.mouseCoordinates = coords;
    },

    updateActualSizes(w, h) {
      const image = new Image();
      image.src = this.currentImage.src;
      image.width = w;
      image.height = h;
      this.currentImage = image;
    },
    updateWidth(w) {
      this.updateActualSizes(w, this.currentImage.height)
    },
    updateHeight(h) {
      this.updateActualSizes(this.currentImage.width, h)
    },
  },

  computed: {
    hasImg() {
      return !!this.currentImage?.src;
    }
  },
};
</script>

<template>
  <aside>
    <SideTools :state="state" :hasImg="hasImg" :scale="scale" @changeState="changeState"
      @updateScale="(value) => (scale = value)" />
    <ColorPicker v-if="state === 'pipette'" :pickedColor="pickedColor" :mouseCoordinates="mouseCoordinates"
      :isShiftPressed="isShiftPressed" />
    <EditImage v-if="state === 'edit'" :interpolation="interpolation"
      @updateInterpolation="(value) => interpolation = value" :width="currentImage.width" @updateWidth="updateWidth"
      :height="currentImage.height" @updateHeight="updateHeight" :ogWidth="currentImage.naturalWidth"
      :ogHeight="currentImage.naturalHeight" />
    <StatusBar v-if="hasImg" :state="state" :ih="currentImage.height" :iw="currentImage.width" :scale="scale" />
    <!-- :pickedColor="pickedColor" :xMouse="mouseCoordinates[0]" :yMouse="mouseCoordinates[1]" -->
  </aside>
  <main>
    <MainCanvas :state="state" :currentImg="currentImage" :iw="iw" :ih="ih" :pickedColor="pickedColor" :scale="scale"
      :isShiftPressed="isShiftPressed" :interpolation="interpolation" @updateImageSizes="updateImageSizes"
      @updateColor="(color) => (pickedColor = color)" @updateCoordinates="updateCoordinates" @changeState="changeState"
      @updateImg="updateImage" />
  </main>
</template>

<style scoped>
aside,
main {
  height: calc(100vh - 2rem * 2 - 10vh);
}

aside {
  float: right;
  width: 25%;
  /* height: 100%; */
  margin-left: 5%;

  display: flex;
  flex-direction: column;
  gap: 1rem;
  padding: 1rem;

  justify-content: space-between;
}

main {
  width: 70%;
}
</style>