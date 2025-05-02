<script lang="ts" setup>
defineProps({
  state: String,
  ih: Number,
  iw: Number,
  pickedColor: String,
  xMouse: Number,
  yMouse: Number,
  scale: Number,
});
</script>

<template>
  <div>
    <em v-if="state">Now working with: {{ state }}</em>
    <div class="status-bar">
      <span v-if="scale">Scale: {{ scale }}</span>
      <span v-if="iw && ih">Actual size: {{ iw }}&nbsp;x&nbsp;{{ ih }}</span>
      <span v-if="state === 'pipette' && pickedColor">Color: <span class="pipette-color" :style="{
        background: pickedColor,
      }"></span>&nbsp;{{ pickedColor }} </span>
      <span v-if="state === 'pipette' && pickedColor && xMouse && yMouse">
        Color coordinates: ({{ xMouse }},&nbsp;{{ yMouse }})
      </span>
    </div>
  </div>
</template>


<style>
.status-bar {
  display: flex;
  flex-direction: column;
  /* align-items: center; */
  /* justify-content: center; */
  gap: 0.2rem;
  /* padding: 0.5rem; */
}

.pipette-color {
  display: inline-block;
  width: 15px;
  height: 15px;
}
</style>