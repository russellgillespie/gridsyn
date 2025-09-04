<template>
  <div class="lcd-top-bar">
    <div class="lcd-field project">{{ project }}</div>
    <div class="lcd-field midi">
      <strong>MIDI:</strong> <span>{{ midiInput }}</span>
    </div>
    <div class="lcd-field channel">
      <strong>CH:</strong> <span>{{ midiChannel }}</span>
    </div>
    <div class="lcd-field tempo">
      <strong>TEMPO:</strong> <span>{{ tempo }} BPM</span>
    </div>
  </div>
</template>
<script>
export default {
  props: {
    project: String,
    midiInput: String,
    midiChannel: Number,
    tempo: Number
  }
};
</script>
<style scoped>
.lcd-top-bar {
  font-family: 'Nova Mono', 'Menlo', monospace;
  display: flex;
  justify-content: space-evenly;
  align-items: center;
  font-size: 1.01rem;
  background: linear-gradient(90deg, #28281b 80%, #4b482b 100%);
  border-bottom: 3.5px solid #dbe518;
  color: #e5ff70;
  letter-spacing: 0.05em;
  text-shadow: 0 2px 2px #000a;
  min-height: 44px;
}
.lcd-field {
  margin: 0 1.2vw;
  white-space: nowrap;
}
</style>
