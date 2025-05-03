<script lang="ts">
export default {
  name: 'SideTools',

  props: {
    state: String,
    hasImg: Boolean,
    scale: Number,
  },
  emits: [
    'changeState',
    'updateScale',
  ],
  watch: {
    state(newVal, oldVal) {
      const parent = (this.$refs.tools as HTMLElement | null);
      parent?.querySelector<HTMLButtonElement>(`.button[value=${oldVal || 'null'}]`)?.classList.remove('active');
      parent?.querySelector<HTMLButtonElement>(`.button[value=${newVal || 'null'}]`)?.classList.add('active');
    },
  },
  methods: {
    handleButtonClick(event: MouseEvent) {
      const target = event.target as HTMLButtonElement;
      if (!target) return;

      const value = target.value;
      this.$emit('changeState', value === this.state ? '' : value);
    },
    handleScaleChange(event: Event) {
      const target = event.target as HTMLInputElement | null;
      if (!target) return;
      const value = target.value;
      this.$emit('updateScale', value);
    },
  },
};
</script>

<template>
  <div class="side-tools" ref="tools">

    <button class="button" @click="handleButtonClick" :disabled="!hasImg" value="hand"
      title="Move the image across the canvas">
      Hand
    </button>
    <button class="button" @click="handleButtonClick" :disabled="!hasImg" value="pipette"
      title="Use the pipette tool to select colors from the image">
      Pipette
    </button>
    <button class="button" @click="handleButtonClick" :disabled="!hasImg" value="edit" title="Edit the image">
      Edit
    </button>
    <button class="button" @click="handleButtonClick" :disabled="!hasImg" value="save" title="Save current image">
      Save
    </button>
    <!-- reset img button -->
    <button class="button" @click="handleButtonClick" :disabled="!hasImg" value="close"
      title="Stop working with current image">
      Close
      <!-- TODO: add modal "are you sure" -->
    </button>

    <label class="slider">
      <span class="scale">Scale</span>
      10&nbsp;<input type="range" class="level" :disabled="!hasImg" :min="10" :max="300" :step="10" :value="scale"
        @change="handleScaleChange" />&nbsp;300
      <span class="scale">{{ scale }}</span>
    </label>
  </div>
</template>

<style scoped>
.button {
  padding: 10px;
  margin: 10px;
  /* font-size: 30px; */
}

.button.active {
  background-color: var(--color-blue);
  /* color: #000; */
}

.slider {
  /* slider */
  --slider-width: 70%;
  --slider-height: 6px;
  --slider-bg: rgb(82, 82, 82);
  --slider-border-radius: 999px;
  /* level */
  --level-color: #fff;
  --level-transition-duration: .1s;
  /* icon */
  --icon-margin: 15px;
  --icon-color: var(--slider-bg);
  --icon-size: 25px;
}

.slider {
  cursor: pointer;
}

.slider .level {
  display: inline-block;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  width: var(--slider-width);
  height: var(--slider-height);
  background: var(--slider-bg);
  overflow: hidden;
  border-radius: var(--slider-border-radius);
  -webkit-transition: height var(--level-transition-duration);
  -o-transition: height var(--level-transition-duration);
  transition: height var(--level-transition-duration);
  cursor: inherit;
}

.slider .level::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 0;
  height: 0;
  -webkit-box-shadow: -200px 0 0 200px var(--level-color);
  box-shadow: -200px 0 0 200px var(--level-color);
}

.slider:hover .level {
  height: calc(var(--slider-height) * 2);
}

.scale {
  display: block;
  text-align: center;
}
</style>
