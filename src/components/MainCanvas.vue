<script lang="ts">
import { defineComponent } from 'vue';
import UploadImg from './UploadImg.vue';

const DEFAULT_OFFSET = 100; // doubled (sum of offsets)
const CHECK_PADDING = 10; // to not be far from

export default defineComponent({
  name: 'MainCanvas',
  components: { UploadImg },
  props: {
    state: String,

    currentImg: Image,

    scale: Number,
    iw: Number,
    ih: Number,
    isShowCorrection: Boolean,
    isShowFiltration: Boolean,

    interpolation: String,
    isShiftPressed: Boolean,
  },
  emits: [
    'changeState',
    'updateImg',
    'updateImageSizes',
    'updateColor',
    'updateCoordinates',
    // TODO
  ],
  data() {
    return {
      // currentImg: new Image(),

      canvasRef: undefined as HTMLCanvasElement | undefined,
      // iw: 0,
      // ih: 0,
      startX: 0,
      startY: 0,
      offsetX: null as number | null,
      offsetY: null as number | null,

      isDragging: false,
    };
  },
  watch: {

    state(newVal) {
      if (newVal === "save") {
        this.saveImage();
        this.$emit('changeState', '')
      }
    },
    currentImg() {
      this.drawImage();
    },
    scale() {
      this.drawImage();
    },
    interpolation() {
      this.drawImage();
    },
    // iw() {
    //   this.drawImage();
    // },
    // ih() {
    //   this.drawImage();
    // },

  },
  mounted() {
    this.canvasRef = this.$refs.canvas as HTMLCanvasElement | undefined;
    window.addEventListener('resize', this.drawImage);

    // TODO: ADD onSave
  },
  beforeUnmount() {
    window.removeEventListener('resize', this.drawImage);

    if (!!this.currentImg?.src) {
      URL.revokeObjectURL(this.currentImg.src);
    }
  },
  methods: {
    handleImageSelected(imageUrl: string) {
      const newImage = new Image();
      newImage.src = imageUrl;
      // newImage.setAttribute('crossOrigin', 'Anonymous');
      // newImage.crossOrigin = `Anonymous`;

      this.$emit('updateImg', newImage);

      // this.drawImage();
    },

    getImageSizes(canvas: HTMLCanvasElement, img: HTMLImageElement): [number, number, number, number] {
      const cw = canvas.width;
      const ch = canvas.height;
      const iw = img.width;
      const ih = img.height;

      const offsetX = this.offsetX;
      const offsetY = this.offsetY;

      const prevIW = this.iw;
      const prevIH = this.ih;

      type SizesRes = [number, number, number, number];

      return getScaledSize(getSizeType(), this.scale);

      function getSizeType(): SizesRes {
        // Все в рамках
        if (iw <= cw && ih <= ch) return getSmaller();
        // Ширина больше; Высота в рамках
        if (cw <= iw && ih <= ch) return getWidthLarger();
        // Высота больше; Ширина в рамках
        if (ch <= ih && iw <= cw) return getHeightLarger();
        // Больше всех рамок
        if (cw < iw && ch < iw) return getAllLarger();

        return [0, 0, 0, 0];
      }

      function getSmaller(): SizesRes {
        const dx = (cw - iw) / 2;
        const dy = (ch - ih) / 2;

        return [iw, ih, dx, dy];
      }

      function getWidthLarger(): SizesRes {
        const coef = cw / iw;
        const heightResize = ih * coef;
        const dx = DEFAULT_OFFSET / 2;
        const dy = Math.abs(heightResize - ch) / 2;

        return [cw - DEFAULT_OFFSET, heightResize, dx, dy];
      }

      function getHeightLarger(): SizesRes {
        const coef = ch / ih;
        const widthResize = iw * coef;
        const dx = Math.abs(widthResize - cw) / 2;
        const dy = DEFAULT_OFFSET / 2;

        return [widthResize, ch - DEFAULT_OFFSET, dx, dy];
      }

      function getAllLarger(): SizesRes {
        if (ih <= iw) return getWidthLarger();
        if (iw < ih) return getHeightLarger();

        return [0, 0, 0, 0];
      }

      function getScaledSize([iw_curr, ih_curr, dx, dy]: [number, number, number, number], scale = 100): [number, number, number, number] {
        const sizeCoef = scale / 100;
        const newiw = ~~(iw_curr * sizeCoef);
        const newih = ~~(ih_curr * sizeCoef);

        const newdx = (offsetX || dx) + ((prevIW ?? newiw) - newiw) / 2;
        const newdy = (offsetY || dy) + ((prevIH ?? newih) - newih) / 2;

        const patchedOffsetX = (newdx + CHECK_PADDING > cw) ? newdx - CHECK_PADDING : ((newdx + newiw < CHECK_PADDING) ? -newiw + CHECK_PADDING : newdx);
        const patchedOffsetY = (newdy + CHECK_PADDING > ch) ? newdy - CHECK_PADDING : ((newdy + newih < CHECK_PADDING) ? -newih + CHECK_PADDING : newdy);

        return [patchedOffsetX, patchedOffsetY, newiw, newih];
      }
    },

    drawImage() {
      const canvas = this.canvasRef;
      const ctx = canvas?.getContext('2d');
      const newImg = this.currentImg;
      if (!canvas || !ctx || !newImg) return;
      newImg.onload = () => {
        canvas.width = canvas.clientWidth;
        canvas.height = canvas.clientHeight;

        const [dx, dy, iw, ih] = this.getImageSizes(canvas, newImg);

        this.offsetX = dx;
        this.offsetY = dy;

        if (this.offsetX === null || this.offsetY === null) return;

        ctx.clearRect(0, 0, canvas.width, canvas.height);

        if (this.interpolation === 'nearestNeighbor') {
          ctx.imageSmoothingEnabled = false;
        }
        else {
          ctx.imageSmoothingEnabled = true;
        }

        ctx.drawImage(newImg, this.offsetX, this.offsetY, iw, ih);
        const imageData = ctx.getImageData(this.offsetX, this.offsetY, iw, ih);

        const interpolatedData = this.interpolationCb(imageData, iw, ih);
        if (interpolatedData !== null) {
          ctx.putImageData(interpolatedData, this.offsetX, this.offsetY);
        }
        // else {
        //   ctx.imageSmoothingEnabled = true;
        // }

        this.$emit('updateImageSizes', ~~iw, ~~ih);
      };
      // force onload
      newImg.src = this.currentImg.src;
    },
    interpolationCb(img: ImageData, iw: number, ih: number): ImageData | null {
      if (this.interpolation === 'nearestNeighbor') {
        return this.nearestNeighborInterpolation(img, iw, ih);
      }
      return null;
    },
    nearestNeighborInterpolation(img: ImageData, newWidth: number, newHeight: number): ImageData {
      const originalWidth = img.width;
      const originalHeight = img.height;
      const scaleX = originalWidth / newWidth;
      const scaleY = originalHeight / newHeight;
      const newData = new Uint8ClampedArray(newWidth * newHeight * 4);

      for (let y = 0; y < newHeight; y++) {
        for (let x = 0; x < newWidth; x++) {
          const px = Math.floor(x * scaleX);
          const py = Math.floor(y * scaleY);
          const index = (y * newWidth + x) * 4;
          const originalIndex = (py * originalWidth + px) * 4;

          newData[index] = img.data[originalIndex];
          newData[index + 1] = img.data[originalIndex + 1];
          newData[index + 2] = img.data[originalIndex + 2];
          newData[index + 3] = img.data[originalIndex + 3];
        }
      }
      return new ImageData(newData, newWidth, newHeight);
    },

    moveImage() {
      const canvas = this.canvasRef;
      if (!canvas || !this.iw || !this.ih || this.offsetX === null || this.offsetY === null) return;


      this.drawImage();
    },
    handleColorPick({ offsetX, offsetY }: { offsetX: number; offsetY: number }): string | undefined {
      const ctx = this.canvasRef?.getContext('2d');
      if (!ctx) return;

      // TODO: fix
      console.log(this.currentImg?.crossOrigin);
      const pixel = ctx.getImageData(offsetX, offsetY, 1, 1).data;
      return `rgb(${pixel[0]}, ${pixel[1]}, ${pixel[2]})`;
    },
    handleCoordinates({ offsetX, offsetY, clientX, clientY }: { offsetX: number; offsetY: number; clientX: number; clientY: number }): [number | null, number | null] {
      if (!this.iw || !this.ih) return [null, null];

      const x = clientX - this.startX;
      const y = clientY - this.startY;
      const xMouse = offsetX - x;
      const yMouse = offsetY - y;

      if (xMouse <= 0 || yMouse <= 0 || this.iw <= xMouse || this.ih <= yMouse) {
        return [null, null];
      }
      return [~~xMouse, ~~yMouse];
    },
    saveImage(): void {
      const imageDataURL = this.canvasRef?.toDataURL('image/png');
      if (!imageDataURL) return;
      const link = document.createElement('a');
      link.href = imageDataURL;
      link.download = 'image_edited.png';
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },

    handleMouseDown(e: MouseEvent) {
      this.isDragging = true;
      this.startX = e.clientX - (this.offsetX ?? 0);
      this.startY = e.clientY - (this.offsetY ?? 0);

      if (this.state == 'pipette') {
        this.$emit('updateColor', this.handleColorPick(e));
        this.$emit('updateCoordinates', this.handleCoordinates(e));
      }
    },
    handleMouseUp(): void {
      this.isDragging = false;
    },
    handleMouseMove(e: MouseEvent): void {
      if (this.state === 'hand' && this.isDragging) {
        const x = e.clientX - this.startX;
        const y = e.clientY - this.startY;
        this.offsetX = x;
        this.offsetY = y;
        this.moveImage();
      }
    },
    handleMouseWheel(event: WheelEvent): void {
      event.preventDefault();
      const delta = Math.sign(event.deltaY);

      if (this.offsetX === null) this.offsetX = 0;
      if (this.offsetY === null) this.offsetY = 0;

      if (this.isShiftPressed) {
        this.offsetX -= delta * 7;
      } else {
        this.offsetY += delta * 7;
      }

      this.moveImage();
    },
  },
});
</script>

<template>
  <UploadImg v-show="!currentImg?.src" @onImageSelected="handleImageSelected" />

  <canvas v-show="!!currentImg?.src" ref="canvas" :class="'canvas ' + state" id="canvas" tabindex="0"
    @mousedown="handleMouseDown" @mousemove="handleMouseMove" @mouseup="handleMouseUp" @wheel="handleMouseWheel">
  </canvas>
</template>

<style scoped>
.canvas {
  widows: 100%;
  width: 100%;
  height: 100%;
  /* cursor: crosshair; */
  background: linear-gradient(45deg,
      rgba(255, 255, 255, 0.0980392) 25%,
      transparent 25%,
      transparent 75%,
      rgba(255, 255, 255, 0.0980392) 75%,
      rgba(255, 255, 255, 0.0980392) 0),
    linear-gradient(45deg,
      rgba(255, 255, 255, 0.0980392) 25%,
      transparent 25%,
      transparent 75%,
      rgba(255, 255, 255, 0.0980392) 75%,
      rgba(255, 255, 255, 0.0980392) 0),
    black;
  background-repeat: repeat, repeat;
  background-position: 0px 0, 10px 10px;
  transform-origin: 0 0 0;
  background-size: 20px 20px, 20px 20px;

  border: 2px solid currentColor;
  border-radius: 10px;
}

.canvas.pipette {
  /* TODO: fix icon to be precise */
  --pipette-icon: url(data:image/x-icon;base64,AAABAAEAEBAAAAAAIABoBAAAFgAAACgAAAAQAAAAIAAAAAEAIAAAAAAAQAQAAAAAAAAAAAAAAAAAAAAAAAD///8BAAAAfwAAAIH///8B////Af///wH///8B////Af///wH///8B////Af///wH///8B////Af///wH///8BAAAAcQAAAN8AAADTAAAArwAAAE8AAAAJ////Af///wH///8B////Af///wH///8B////Af///wH///8B////AQAAAJEAAADLAAAAEwAAAHsAAADDAAAA3wAAADP///8B////Af///wH///8B////Af///wH///8B////Af///wH///8BAAAAuQAAAHX///8B////AQAAAF0AAADtAAAAOf///wH///8B////Af///wH///8B////Af///wH///8B////AQAAAFMAAAC/////Af///wH///8BAAAAUQAAAO0AAAA5////Af///wH///8B////Af///wH///8B////Af///wEAAAANAAAA5wAAAFH///8B////Af///wEAAABRAAAA7QAAADn///8B////Af///wH///8B////Af///wH///8B////AQAAAD0AAADtAAAARf///wH///8B////AQAAAFEAAADtAAAAOQAAACP///8B////Af///wH///8B////Af///wH///8BAAAARQAAAO0AAABF////Af///wH///8BAAAAUQAAAO0AAADvAAAAmf///wH///8B////Af///wH///8B////Af///wEAAABFAAAA7QAAAEX///8B////AQAAADUAAADxAAAA/wAAAPcAAAAr////Af///wH///8B////Af///wH///8B////AQAAAEUAAADtAAAARQAAADUAAADvAAAA/wAAAPcAAABFAAAALf///wH///8B////Af///wH///8B////Af///wH///8BAAAARQAAAO0AAADvAAAA/wAAAPcAAABFAAAAgwAAAPsAAABX////Af///wH///8B////Af///wH///8B////AQAAACUAAADvAAAA/wAAAPcAAABFAAAAgwAAAP8AAAD/AAAA+wAAAE////8B////Af///wH///8B////Af///wEAAAADAAAAoQAAAPcAAABFAAAAgwAAAP8AAAD/AAAA/wAAAP8AAADf////Af///wH///8B////Af///wH///8B////AQAAAAMAAAAvAAAALQAAAPsAAAD/AAAA/wAAAP8AAAD/AAAA+////wH///8B////Af///wH///8B////Af///wH///8B////Af///wEAAABXAAAA+wAAAP8AAAD/AAAA/wAAALX///8B////Af///wH///8B////Af///wH///8B////Af///wH///8B////AQAAAE8AAADfAAAA+wAAALUAAAAXAAD//wAA//8AAP//AAD//wAA//8AAP//AAD//wAA//8AAP//AAD//wAA//8AAP//AAD//wAA//8AAP//AAD//w==);
  cursor: var(--pipette-icon), crosshair;
  /* cursor: crosshair */
}

.canvas.hand {
  cursor: grab;
}

.canvas.hand:active {
  cursor: grabbing;
}
</style>
