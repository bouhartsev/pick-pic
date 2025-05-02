<script lang="ts">

type ColorData = Partial<{
  rgb: string; // rgb(0, 0, 0)
  xyz: string; // xyz(0, 0, 0)
  lab: string; // lab(0, 0, 0)
  coordinates: string; // (0, 0)
}>

export default {
  name: 'ColorPicker',
  props: {
    pickedColor: String,
    mouseCoordinates: Array<number>,
    isShiftPressed: Boolean,
  },
  data() {
    return {
      colors: [{}, {}] as Array<ColorData>,
    };
  },
  computed: {
    currentColorIndex() {
      return this.isShiftPressed ? 1 : 0;
    },
    contrastRatio() {

      if (!this.colors[0].rgb || !this.colors[1].rgb) return 0;


      const sRGBToLinear = (c: number) => {
        c = c / 255;
        return c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4);
      };

      const getLuminance = (rgb: number[]) => {
        const r = sRGBToLinear(rgb[0]);
        const g = sRGBToLinear(rgb[1]);
        const b = sRGBToLinear(rgb[2]);
        return 0.2126 * r + 0.7152 * g + 0.0722 * b;
      };

      const luminances = this.colors.map(({ rgb }) => {
        if (!rgb) return 0;
        return getLuminance(this.parseRGB(rgb));
      });

      const maxLuminance = Math.max(...luminances);
      const minLuminance = Math.min(...luminances);

      const contrastRatio = (maxLuminance + 0.05) / (minLuminance + 0.05);

      return Math.round(contrastRatio * 100) / 100; // round to 2 decimal places
    }
  },
  watch: {
    pickedColor(newValue) {
      this.colors[this.currentColorIndex] = { ...this.colors[this.currentColorIndex], ...this.getColorData(newValue) };
    },
    mouseCoordinates(newValue) {
      this.colors[this.currentColorIndex] = { ...this.colors[this.currentColorIndex], coordinates: `(${newValue.join(', ')})` };
    },
  },
  methods: {
    getColorData(color: string) {
      const parsedRGB = this.parseRGB(color);
      const xyz = this.rgbToXyz(parsedRGB);
      const lab = this.xyzToLab(xyz);
      return {
        rgb: color,
        xyz: `xyz(${this.formatColor(xyz)})`,
        lab: `lab(${this.formatColor(lab)})`,
      };
    },
    parseRGB(input: string): number[] {
      return input
        .split("(")[1]
        .split(")")[0]
        .split(",")
        .map((x) => +x);
    },

    parseHEX(input: string): number[] {

      const collen = (input.length - 1) / 3;
      const fact = [17, 1, 0.062272][collen - 1];
      return [
        Math.round(parseInt(input.substr(1, collen), 16) * fact),
        Math.round(parseInt(input.substr(1 + collen, collen), 16) * fact),
        Math.round(parseInt(input.substr(1 + 2 * collen, collen), 16) * fact),
      ];
    },

    rgbToXyz(rgb: number[]): number[] {
      let r = rgb[0] / 255;
      let g = rgb[1] / 255;
      let b = rgb[2] / 255;

      // Применяем коррекцию для RGB пространства
      r = r > 0.04045 ? Math.pow((r + 0.055) / 1.055, 2.4) : r / 12.92;
      g = g > 0.04045 ? Math.pow((g + 0.055) / 1.055, 2.4) : g / 12.92;
      b = b > 0.04045 ? Math.pow((b + 0.055) / 1.055, 2.4) : b / 12.92;

      // Применяем коэффициенты преобразования
      r *= 100;
      g *= 100;
      b *= 100;

      // Вычисляем XYZ
      const x = r * 0.4124 + g * 0.3576 + b * 0.1805;
      const y = r * 0.2126 + g * 0.7152 + b * 0.0722;
      const z = r * 0.0193 + g * 0.1192 + b * 0.9505;

      return [x, y, z];
    },

    xyzToLab(xyz: number[]): number[] {
      const [x, y, z] = xyz;

      // Коэффициенты для преобразования
      const xn = 95.047;
      const yn = 100.0;
      const zn = 108.883;

      const fx = x / xn;
      const fy = y / yn;
      const fz = z / zn;

      const epsilon = 0.008856;
      const kappa = 903.3;

      const f = (t: number) =>
        t > epsilon ? Math.pow(t, 1 / 3) : (kappa * t + 16) / 116;

      const L = 116 * f(fy) - 16;
      const a = 500 * (f(fx) - f(fy));
      const b = 200 * (f(fy) - f(fz));

      return [L, a, b];
    },

    formatColor(arr: number[]): string {
      return arr.map((x) => x.toFixed(2)).join(", ");
    },

  },


};
</script>

<template>
  <div>
    <h3>Pipette colors:</h3>

    <ul>
      <li v-for="color in colors" :key="color.coordinates">
        <span class="pipette-color" :style="{ background: color.rgb }"></span>
        <span>{{ color.rgb }}</span>
        <div>{{ color.xyz }}</div>
        <div>{{ color.lab }}</div>
        <div>{{ color.coordinates }}</div>
      </li>
    </ul>
    <div v-show="contrastRatio !== 0">
      <h4>Contrast ratio:</h4>
      {{ contrastRatio }}:1 <span v-if="contrastRatio < 4.5">- NOT ENOUGH</span>
    </div>
  </div>
</template>

<style>
.pipette-color {
  display: inline-block;
  width: 15px;
  height: 15px;
}

ul {
  list-style: none;
  padding: 0;
}

ul>li>span {
  margin-right: 0.5em;
}

ul>li {
  margin-bottom: 0.5em;
}
</style>
