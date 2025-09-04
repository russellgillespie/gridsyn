<template>
  <div class="viewport-centered">
    <div class="safe-zone">
      <!-- Floating toggle button -->
      <button
        class="floating-toggle"
        @click="toggleDetails"
        :aria-label="detailsMode ? 'Close Details View' : 'Open Details View'"
      >
        <span v-if="!detailsMode">☰</span>
        <span v-else>✕</span>
      </button>
      <div class="grid-and-panels" :class="{ 'details-active': detailsMode }" role="region">
        <!-- Top LCD Inspector -->
        <div v-if="detailsMode" class="panel toolbar top lcd-top-bar">
          <div class="lcd-field project">{{ project }}</div>
          <div class="lcd-field midi">
            <strong>MIDI:</strong> <span>{{ midiInputName }}</span>
          </div>
          <div class="lcd-field channel">
            <strong>CH:</strong> <span>{{ midiChannel }}</span>
          </div>
          <div class="lcd-field tempo">
            <strong>TEMPO:</strong> <span>{{ tempo }} BPM</span>
          </div>
        </div>
        <!-- Left Inspector (Prev Selected Cell) -->
        <div v-if="detailsMode" class="panel inspector left insp-content">
          <div v-if="previousCell">
            <div class="insp-title">PREV: <b>{{ previousCell.name }}</b></div>
            <div class="insp-param-list">
              <div v-for="(val, key) in previousCell.params" :key="key" class="insp-param">
                <span class="p-label">{{ key }}</span>
                <span class="p-value">{{ val }}</span>
              </div>
            </div>
          </div>
          <div class="insp-empty" v-else>
            <span>No Prev Module</span>
          </div>
        </div>
        <!-- Right Inspector (Current Selected Cell) -->
        <div v-if="detailsMode" class="panel inspector right insp-content">
          <div v-if="currentCell">
            <div class="insp-title">CURR: <b>{{ currentCell.name }}</b></div>
            <div class="insp-param-list">
              <div v-for="(val, key) in currentCell.params" :key="key" class="insp-param">
                <span class="p-label">{{ key }}</span>
                <span class="p-value">{{ val }}</span>
              </div>
            </div>
          </div>
          <div class="insp-empty" v-else>
            <span>No Module Selected</span>
          </div>
        </div>
        <!-- Bottom Smart Knobs Bar -->
        <div v-if="detailsMode" class="panel toolbar bottom smart-knobs-bar">
          <div
            v-for="(knob, i) in smartKnobs"
            :key="knob.id"
            class="smart-knob"
          >
            <label class="kn-name">{{ knob.name }}</label>
            <input
              type="range"
              min="0"
              max="1"
              step="0.01"
              :value="knob.id === 'master' ? masterVolume : knob.value"
              @input="onKnobChange(knob.id, $event.target.value)"
            />
          </div>
        </div>
        <!-- Central grid: either normal grid or large expert cell -->
        <main
          class="fullscreen-grid"
          :class="{ 'grid-shrink': detailsMode, 'grid-expert': expertCellIndex !== null }"
          key="main-grid"
        >
          <template v-if="expertCellIndex === null">
            <div
              v-for="(cell, idx) in gridCells"
              :key="cell.id"
              class="grid-cell"
              :class="{
                selected: selectedIndex === idx,
                prevSelected: prevSelectedIndex === idx
              }"
              :style="{ borderColor: cell.color }"
              tabindex="0"
              @click="handleCellSelect(idx)"
            >
              <div class="cell-content">
                <div class="mod-label">{{ cell.name }}</div>
                <div class="cell-led" :style="{ background: cell.color }"></div>
              </div>
              <button
                class="expert-btn"
                :aria-label="'Show Expert View for ' + cell.name"
                @click.stop="openExpert(idx)"
                tabindex="0"
              >🔎</button>
            </div>
          </template>
          <template v-else>
            <div
              class="grid-cell expert-cell"
              :style="{ borderColor: gridCells[expertCellIndex].color }"
              tabindex="0"
            >
              <button
                class="expert-btn expert-close"
                :aria-label="'Close Expert View for ' + gridCells[expertCellIndex].name"
                @click.stop="closeExpert"
                tabindex="0"
              >✕</button>
              <div class="cell-content expert-body">
                <div class="mod-label expert-title">
                  {{ gridCells[expertCellIndex].name }} — Expert View
                </div>
                <div style="height:2vw"/>
                <div v-for="(val, key) in gridCells[expertCellIndex].params"
                  :key="key"
                  style="font-family:monospace;margin:1vw 0;font-size:1.2em;">
                  <b>{{ key }}:</b> {{ val }}
                </div>
                <div style="height:2vw"/>
                <div
                  style="font-size:1.1em;color:#67ffc0;opacity:.72;text-align:center;">
                  [ Put advanced editing UI here! ]
                </div>
              </div>
            </div>
          </template>
        </main>
      </div>
    </div>
  </div>
</template>

<script>
const defaultGrid = [
  { id: 0, name: "Oscillator", color: "#50E3C2", params: { wave: "saw", octave: 1 } },
  { id: 1, name: "Filter", color: "#B8E986", params: { cutoff: 1200, res: 0.5 } },
  { id: 2, name: "Envelope", color: "#F8E71C", params: { attack: 0.2, release: 0.8 } },
  { id: 3, name: "LFO", color: "#417505", params: { rate: 4, depth: 0.8 } },
  { id: 4, name: "Mixer", color: "#BD10E0", params: { balance: 0.5 } },
  { id: 5, name: "FX", color: "#EB4D4B", params: { type: "delay", wet: 0.35 } },
  { id: 6, name: "Sequencer", color: "#4A90E2", params: { steps: 16 } },
  { id: 7, name: "Arpeggiator", color: "#FF6F61", params: { mode: "up", rate: "1/8" } },
  { id: 9, name: "Compressor", color: "#D0A441", params: { thresh: -20, ratio: 3 } },
  { id: 10, name: "Delay", color: "#EB4D4B", params: { type: "delay", wet: 0.35 } },
  { id: 11, name: "Reverb", color: "#4A90E2", params: { steps: 16 } },
  { id: 12, name: "Drive", color: "#FF6F61", params: { mode: "up", rate: "1/8" } },
  { id: 13, name: "PHAT", color: "#FF6F61", params: { mode: "up", rate: "1/8" } },
  { id: 14, name: "Feedback", color: "#D0A441", params: { thresh: -20, ratio: 3 } },
  { id: 15, name: "Master", color: "#D0A441", params: { thresh: -20, ratio: 3 } }
];

const initialSmartKnobs = [
  { name: "Master Vol", id: "master", value: 0.75, midiCC: 7 },
  { name: "Tone", id: "tone", value: 0.6, midiCC: 74 },
  { name: "Drive", id: "drive", value: 0.2, midiCC: 71 },
  { name: "Mod", id: "mod", value: 0.2, midiCC: 1 },
  { name: "Length", id: "length", value: 0.4, midiCC: 73 },
  { name: "Space", id: "space", value: 0.4, midiCC: 91 },
  { name: "FX", id: "fx", value: 0.2, midiCC: 92 },
  { name: "Release", id: "rel", value: 0.6, midiCC: 72 }
];

export default {
  name: "App",
  data() {
    return {
      detailsMode: false,
      selectedIndex: 4, // center startup
      prevSelectedIndex: null,
      gridCells: defaultGrid,
      project: "Medeski Jam 01",
      midiInputName: "No Device",
      midiChannel: 1,
      tempo: 120,
      smartKnobs: initialSmartKnobs,
      masterVolume: 0.75,
      expertCellIndex: null
    };
  },
  computed: {
    previousCell() {
      return typeof this.prevSelectedIndex === 'number' && this.prevSelectedIndex !== null
        ? this.gridCells[this.prevSelectedIndex]
        : null;
    },
    currentCell() {
      return this.gridCells[this.selectedIndex];
    }
  },
  methods: {
    toggleDetails() {
      this.detailsMode = !this.detailsMode;
      this.expertCellIndex = null; // auto-close expert if toggling views
    },
    handleCellSelect(idx) {
      if (idx !== this.selectedIndex) {
        this.prevSelectedIndex = this.selectedIndex;
        this.selectedIndex = idx;
      }
    },
    onKnobChange(id, value) {
      if (id === "master") this.masterVolume = parseFloat(value);
    },
    openExpert(idx) {
      this.expertCellIndex = idx;
    },
    closeExpert() {
      this.expertCellIndex = null;
    }
  }
};
</script>

<style>
html, body, #app {
  height: 100dvh;
  width: 100vw;
  margin: 0;
  padding: 0;
  background: #111;
  overflow: hidden;
}
.viewport-centered {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100dvh;
  width: 100vw;
  background: #111;
}
.safe-zone {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: min(4vw, 4vh);
  background: linear-gradient(145deg, #232323 70%, #181822);
  border: 6px solid #222;
  border-radius: 2.5vw;
  box-shadow: 0 0 0 6px #161616, 0 8px 32px 6px rgba(0,0,0,0.7);
  min-width: 0;
  min-height: 0;
  box-sizing: border-box;
  overflow: visible;
}
.floating-toggle {
  position: absolute;
  top: 1vw;
  right: 1vw;
  background: #212237ec;
  color: #68ffae;
  border: 2px solid #0a0a0e;
  border-radius: 100px;
  box-shadow: 0 2px 12px #000b;
  font-size: 2rem;
  width: 3.5rem;
  height: 3.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 200;
  cursor: pointer;
  outline: none;
  transition: background 0.17s, border 0.17s;
}
.floating-toggle:active {
  background: #29285f;
  color: #fff;
}
.grid-and-panels {
  display: grid;
  position: relative;
  box-sizing: border-box;
  width: 80vw;
  height: 80vw;
  max-width: 80dvh;
  max-height: 80dvh;
  min-width: 320px;
  min-height: 320px;
  grid-template-columns: 120px 1fr 120px;
  grid-template-rows: 60px 1fr 88px;
  grid-template-areas:
    "panel-top    panel-top    panel-top"
    "panel-left   grid         panel-right"
    "panel-bottom panel-bottom panel-bottom";
  transition:
    width 0.36s cubic-bezier(.67,1.3,.36,1),
    height 0.36s cubic-bezier(.67,1.3,.36,1);
}
.panel.toolbar.top    { grid-area: panel-top; }
.panel.toolbar.bottom { grid-area: panel-bottom; }
.panel.inspector.left { grid-area: panel-left; }
.panel.inspector.right{ grid-area: panel-right; }
.fullscreen-grid      { grid-area: grid; height: 100%; width: 100%; position: relative; }
@media (max-width: 650px), (max-height: 600px) {
  .grid-and-panels {
    width: 96vw;
    height: 96vw;
    max-width: 96dvh;
    max-height: 96dvh;
    grid-template-columns: 60px 1fr 60px;
    grid-template-rows: 36px 1fr 62px;
  }
}
/* All panels: solid background, never overlap grid */
.panel {
  box-shadow: 0 3px 24px #0009;
  border-radius: 16px;
  border: 2px solid #1c1c29;
  user-select: none;
  font-size: 1.13rem;
  font-weight: 500;
  color: #b0ffe7;
  background: #1c1c20;
  opacity: 1 !important;
  pointer-events: auto;
  z-index: 1;
  display: flex;
  align-items: flex-start;
  justify-content: flex-start;
}
.toolbar {
  padding: 0 1.4rem;
  background: #28281b;
  border-bottom: 3.5px solid #dbe518;
  color: #e5ff70;
  align-items: center;
  justify-content: space-between;
}
.smart-knobs-bar {
  flex-direction: row;
  align-items: flex-end;
  gap: 2vw;
  background: #232E36;
  border-top: 3px solid #b2e374d6;
  height: 88px;
  min-height: 60px;
  padding: 0 0.7vw;
  display: flex;
}
.lcd-top-bar {
  font-family: 'Nova Mono', 'Menlo', monospace;
  font-size: 1.01rem;
  background: #28281b;
  border-bottom: 3.5px solid #dbe518;
  letter-spacing: 0.05em;
  text-shadow: 0 2px 2px #000a;
  min-height: 44px;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-evenly;
}
.lcd-field { margin: 0 1vw; white-space: nowrap; }
.insp-content {
  flex-direction: column;
  align-items: flex-start;
  justify-content: flex-start;
  padding: 0.9em 0.65em;
  width: 100%;
  display: flex;
}
.insp-title {
  font-size: 1.02rem;
  color: #faffb4;
  font-family: 'Menlo', monospace;
  margin-bottom: 0.5em;
  text-shadow: 0 2px 3px #0009;
  font-weight: 600;
}
.insp-param-list {display: flex; flex-direction: column; gap: 0.18em;}
.insp-param { display: flex; justify-content: space-between; align-items: center; font-size: 0.94em; color: #dafbfa;}
.p-label { color: #f0d485; margin-right: 0.5em; }
.p-value { color: #b8ffce; }
.insp-empty { color: #686; opacity: 0.5; font-size: 0.85em; text-align: center; margin-top: 1em; }
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
.smart-knob:first-child input[type="range"] { accent-color: #ee6; }
.fullscreen-grid {
  margin: 0;
  box-sizing: border-box;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 1fr);
  gap: 2vw;
  border-radius: 1.5vw;
  width: 100%;
  height: 100%;
  transition:
    filter 0.28s cubic-bezier(.4,1.2,.4,1),
    transform 0.44s cubic-bezier(.67,1.3,.36,1);
}
.fullscreen-grid.grid-shrink {
  /* Only shrink, no blur, no brightness/opacity overlay! */
  filter: none;
  transform: scale(0.81);
}
.fullscreen-grid.grid-expert {
  /* One cell fills grid area */
  display: flex;
  align-items: stretch;
  justify-content: stretch;
  grid-template-columns: none !important;
  grid-template-rows: none !important;
}
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
  position: relative;
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
.expert-btn {
  position: absolute;
  top: 0.4vw;
  right: 0.6vw;
  background: #181c28;
  border: 1.5px solid #333f;
  border-radius: 2em;
  padding: 0 0.46em;
  color: #57ffc3;
  font-size: 1.1em;
  font-family: inherit;
  cursor: pointer;
  z-index: 50;
  transition: background .16s, color .14s;
  outline: none;
}
.expert-btn:active,
.expert-btn:focus {
  background: #67ffc3;
  color: #000;
}
.expert-btn.expert-close {
  background: #222;
  color: #ec5e5e;
  border: 2px solid #ec5e5e47;
  right: 0.95vw;
  top: 0.85vw;
  font-size: 1.34em;
}
.expert-cell {
  width: 100%;
  height: 100%;
  min-width: 0; min-height: 0;
  border-width: 5px;
  border-radius: 1.1vw;
  display: flex !important;
  align-items: stretch !important;
  justify-content: stretch !important;
  background: #242c39;
  padding: min(2vw, 16px);
}
.cell-content.expert-body {
  flex: 1 1 auto;
  justify-content: center;
  align-items: center;
  display: flex;
  flex-direction: column;
}
.expert-title {
  font-size: 2.35em;
  color: #fff;
  font-family: 'Menlo', monospace;
  margin-bottom: 1vw;
  text-shadow: 0 2px 7px #000c;
}
@media (max-width: 650px), (max-height: 600px) {
  .grid-and-panels {
    width: 96vw;
    height: 96vw;
    max-width: 96dvh;
    max-height: 96dvh;
    grid-template-columns: 60px 1fr 60px;
    grid-template-rows: 36px 1fr 62px;
  }
  .expert-title { font-size: 1.15em; }
}
</style>
