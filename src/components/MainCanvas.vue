<script lang="ts">
import { defineComponent } from 'vue';
import UploadImg from './UploadImg.vue';

export default defineComponent({
  name: 'MainCanvas',
  components: { UploadImg },
  data() {
    return {
      img: new Image(),

      canvasRef: null,
      iw: 0,
      ih: 0,
      startX: 0,
      startY: 0,
      offsetX: 0,
      offsetY: 0,
      scale: 100,
      interpolation: '',
    };
  },
  methods: {
    handleImageSelected(imageUrl: string) {
      this.img = new Image();
      this.img.src = imageUrl;
      console.log(this.img);

      this.drawImage(this.img);
    },

    getImageSizes(canvas, img) {
      const offset = 100;
      const cw = canvas.width;
      const ch = canvas.height;
      const iw = img.width;
      const ih = img.height;
      console.log(getSizeType());

      return getScaledSize(canvas, getSizeType(), this.scale);

      function getSizeType() {
        // Все в рамках
        if (iw <= cw && ih <= ch) return getSmaller();
        // Ширина больше; Высота в рамках
        if (cw <= iw && ih <= ch) return getWidthLarger();
        // Высота больше; Ширина в рамках
        if (ch <= ih && iw <= cw) return getHeightLarger();
        // Больше всех рамок
        if (cw < iw && ch < iw) return getAllLarger();
      }

      function getSmaller() {
        const dx = (cw - iw) / 2;
        const dy = (ch - ih) / 2;

        return [iw, ih, dx, dy];
      }

      function getWidthLarger() {
        const coef = cw / iw;
        const heightResize = ih * coef;
        const dx = offset / 2;
        const dy = Math.abs(heightResize - ch) / 2;

        return [cw - offset, heightResize, dx, dy];
      }

      function getHeightLarger() {
        const coef = ch / ih;
        const widthResize = iw * coef;
        const dx = Math.abs(widthResize - cw) / 2;
        const dy = offset / 2;

        return [widthResize, ch - offset, dx, dy];
      }

      function getAllLarger() {
        if (ih <= iw) return getWidthLarger();
        if (iw < ih) return getHeightLarger();
      }

      function getScaledSize({ width, height }, [cw, ch, dx, dy], scale) {
        const sizeCoef = scale / 100;
        const newiw = cw * sizeCoef;
        const newih = ch * sizeCoef;
        const newdx = (width - newiw) / 2;
        const newdy = (height - newih) / 2;

        return [newdx, newdy, newiw, newih];
      }
    },

    drawImage(newImg: HTMLImageElement) {
      const canvas = this.$refs.canvas as HTMLCanvasElement | undefined;
      const ctx = canvas?.getContext('2d');
      if (!canvas || !ctx) return;
      const img = newImg;
      img.onload = () => {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        canvas.width = canvas.clientWidth;
        canvas.height = canvas.clientHeight;
        ctx.imageSmoothingEnabled = false;

        const [dx, dy, iw, ih] = this.getImageSizes(canvas, img);
        console.log(dx, dy, iw, ih);
        const imageData = ctx.getImageData(dx, dy, iw, ih);
        if (imageData instanceof ImageData) {
          const interpolatedData = this.interpolationCb(imageData, ~~iw, ~~ih);
          if (interpolatedData !== null) {
            ctx.putImageData(interpolatedData, dx, dy);
          }
        } else {
          ctx.drawImage(img, dx, dy, iw, ih);
        }

        this.offsetX = dx;
        this.offsetY = dy;
        this.iw = iw;
        this.ih = ih;
        // this.$emit('updateImageSizes', iw, ih)
      };
      // img.src = this.newImg.src
    },
    interpolationCb(img, iw, ih) {
      if (this.interpolation === 'nearestNeighbor') {
        return this.nearestNeighborInterpolation(img, iw, ih);
      }
      return null;
    },
    nearestNeighborInterpolation(img, newWidth, newHeight) {
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
  },
});
</script>

<template>
  <UploadImg @onImageSelected="handleImageSelected" />

  <canvas ref="canvas" class="canvas" id="canvas" tabindex="0">
    <!-- @mousedown="handleMouseDown"
    @mousemove="handleMouseMove"
    @mouseup="handleMouseUp"
    @wheel="handleMouseWheel" -->
  </canvas>
</template>
