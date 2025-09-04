<template>
  <div class="insp-content" v-if="cell">
    <div class="insp-title">{{ label }}: <b>{{ cell.name }}</b></div>
    <div class="insp-param-list">
      <div v-for="(val, key) in cell.params" :key="key" class="insp-param">
        <span class="p-label">{{ key }}</span>
        <span class="p-value">{{ val }}</span>
      </div>
    </div>
  </div>
  <div class="insp-empty" v-else>
    <span>{{ label }} Module</span>
  </div>
</template>
<script>
export default {
  props: {
    cell: Object,
    label: String
  }
};
</script>
<style scoped>
.insp-title {
  font-size: 1.05rem;
  color: #faffb4;
  font-family: 'Menlo', monospace;
  margin-bottom: 0.5em;
  text-shadow: 0 2px 3px #0009;
  font-weight: 600;
}
.insp-param-list {
  display: flex; flex-direction: column; gap: 0.18em;
}
.insp-param {
  display: flex; justify-content: space-between; align-items: center;
  font-size: 0.94em; color: #dafbfa;
}
.p-label { color: #f0d485; margin-right: 0.5em; }
.p-value { color: #b8ffce; }
.insp-empty {
  color: #686;
  opacity: 0.5;
  font-size: 0.85em;
  text-align: center;
  margin-top: 1em;
}
</style>
