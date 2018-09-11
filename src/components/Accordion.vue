<template>
  <div class="accordion">
    <h3 class="accordion-title" @click="open = !open">{{title}}</h3>
    <transition name="slide-fade">
      <div class="accordion-content" v-show="open">
        <slot></slot>
      </div>
    </transition>
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
  /* Enter and leave animations can use different */
  /* durations and timing functions.              */
  .slide-fade-enter-active,
  .slide-fade-leave-active {
    transition: all .3s ease-out;
  }
  /* .slide-fade-leave-active {
    transition: all .3s cubic-bezier(1.0, 0.5, 0.8, 1.0);
  } */
  .slide-fade-enter, .slide-fade-leave-to {
    transform: translateY(-10px);
    opacity: 0;
  }
  .accordion {
    margin-bottom: 10px;
  }
  .accordion-title {
    background: #999;
    color: white;
    margin: 0;
    padding: 10px;
  }
  .accordion-title:hover {
    cursor: pointer;
  }
  .accordion-content {
    border: 1px solid #006245;
    /* display: none; */
    padding: 12px;
  }
  .accordion-content.open {
    display: block;
  }
</style>
