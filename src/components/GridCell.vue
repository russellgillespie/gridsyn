<template>
  <div
    class="grid-cell"
    :class="{ selected: isSelected, prevSelected: isPrevSelected }"
    :style="{ borderColor: cell.color }"
    @click="$emit('select')"
    tabindex="0"
  >
    <div class="cell-content">
      <div class="mod-label">{{ cell.name }}</div>
      <div class="cell-led" :style="{ background: cell.color }"></div>
    </div>
  </div>
</template>
<script>
export default {
  props: {
    cell: Object,
    isSelected: Boolean,
    isPrevSelected: Boolean
  }
};
</script>
<style scoped>
.grid-cell {
  background: #222;
  border: 3px solid #111;
  border-radius: 11px;
  box-shadow: 0 3px 16px #151d, 0 0 0 2px #333 inset;
  overflow: hidden;
  cursor: pointer;
  min-width: 0; min-height: 0;
  outline: none;
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  transition: box-shadow 0.14s, border 0.18s, filter 0.23s;
  user-select: none;
  padding: 0.7vw 0.2vw 0.5vw 0.2vw;
}
.grid-cell.selected {
  border-width: 5px;
  box-shadow: 0 0 0 4px #fff4, 0 0 0 4px var(--cell-color, #fff3);
  filter: brightness(1.12) saturate(1.2);
}
.grid-cell.prevSelected {
  border-style: dashed;
  border-width: 4px;
  filter: grayscale(0.35) opacity(0.7);
}
.mod-label {
  font-size: 0.92rem;
  font-weight: 600;
  font-family: monospace;
  color: #fff;
}
.cell-led {
  margin-top: 13px;
  width: 18px;
  height: 7px;
  border-radius: 2.7px;
  box-shadow: 0 0 6px #fff8, 0 0 12px var(--cell-color, #fff6);
}
</style>
