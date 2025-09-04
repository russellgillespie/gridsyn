<template>
  <div class="smart-knobs-bar">
    <div
      class="smart-knob"
      v-for="(knob, i) in knobs"
      :key="knob.id"
    >
      <label class="kn-name">{{ knob.name }}</label>
      <input
        type="range"
        min="0"
        max="1"
        step="0.01"
        :value="knob.id === 'master' ? masterVolume : knob.value"
        @input="onChange(knob.id, $event.target.value)"
      />
    </div>
  </div>
</template>
<script>
export default {
  props: {
    knobs: Array,
    masterVolume: Number
  },
  methods: {
    onChange(id, val) {
      this.$emit("knobChange", { id, value: parseFloat(val) });
    }
  }
}
</script>
<style scoped>
.smart-knobs-bar {
  display: flex;
  flex-direction: row;
  justify-content: space-evenly;
  align-items: flex-end;
  gap: 2vw;
  background: linear-gradient(180deg, #232E36 83%, #5E8914 98%);
  border-top: 3px solid #b2e374d6;
  height: 88px;
  min-height: 60px;
  padding: 0 0.7vw;
}
.smart-knob {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 42px;
  flex: 1 1 0;
}
.kn-name {
  font-family: monospace; font-size: 0.85em;
  color: #e6facb;
  margin-bottom: 8px;
}
input[type="range"] {
  accent-color: #d8ff44;
  width: 36px; height: 50px;
  writing-mode: bt-lr;
  background: transparent;
  margin: 0;
  cursor: pointer;
}
.smart-knob:first-child input[type="range"] {
  accent-color: #ee6;
}
</style>
