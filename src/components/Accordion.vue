<template>
  <div class="accordion">
    <h3 class="accordion-title" @click="open = !open">{{title}}</h3>
    <div class="accordion-content" :class="{open: open}">
      <slot></slot>
    </div>
  </div>
</template>

<script>
export default {
  props: ['title'],
  data () {
    return {
      open: false
    }
  }
}
</script>

<style>
  .accordion {
    margin-bottom: 10px;
  }
  .accordion-title {
    background: #006245;
    color: white;
    margin: 0;
    padding: 10px;
  }
  .accordion-title:hover {
    cursor: pointer;
  }
  .accordion-content {
    border: 1px solid #006245;
    display: none;
    padding: 12px;
  }
  .accordion-content.open {
    display: block;
  }
</style>
