<script setup>
// Simple two-column layout component that fills available width.
// Provides "left" and "right" named slots.
</script>

<template>
  <div class="column-layout">
    <aside class="left">
      <slot name="left"></slot>
    </aside>

    <section class="right">
      <slot name="right"></slot>
    </section>
  </div>
</template>

<style scoped>
.column-layout {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  align-items: start;
  /* Use viewport width so columns expand to fill the window with a small gutter */
  width: calc(100vw - 2rem);
  max-width: calc(100vw - 2rem);
  margin: 0 auto;
  box-sizing: border-box;
  padding: 0.5rem;
}

/* Make columns stack on small screens */
@media (max-width: 800px) {
  .column-layout {
    grid-template-columns: 1fr;
    width: calc(100vw - 1rem);
    margin: 0 auto;
    gap: 1rem;
    padding: 0.5rem;
  }
}

.left, .right {
  background: #ffffff;
  border: 1px solid #e6e9ef;
  border-radius: 10px;
  padding: 1rem;
  box-shadow: 0 1px 2px rgba(12,15,20,0.03);
  min-height: 200px;
  box-sizing: border-box;
}

/* Slight visual distinction for right column */
.right {
  background: #fbfcff;
}
</style>
