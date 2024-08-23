<script setup>
import { ref } from 'vue'

let props = defineProps({
  type:{
    type: String,
    required: true
  },
  x: {
    type: Number,
    required: true
  },
  y: {
    type: Number,
    required: true
  },
  name: {
    type: String,
    required: true
  },
  color: {
    type: String,
    required: false,
    default: "#FFF"
  },
  content: {
    type: String,
    required: false,
    default: "https://www.mediacollege.com/adobe/premiere/version/pro-cs6/images/CS6_media-offline.png"
  },
  showName: {
    type: Boolean,
    required: false,
    default:  true
  },
  showNode: {
    type: Boolean,
    required: false,
    default:  true
  },
  showContent: {
    type: Boolean,
    required: false,
    default:  (props) =>(props.type == 'image')
  }
})
</script>
<template>
  <g class="node">
    
    <image :x='x-50' :y='y+0' v-if="type == 'image' && showContent" width="100" height="100" :href="content" preserveAspectRatio="none"/>
    <text  :x='x-50' :y='y+0' v-else-if="type == 'text' && showContent">{{content!=''?content:'not provided.'}}</text>
    <circle class="circle" @mouseup="$emit('openPanel', $event)" @mousedown="$emit('startDrag', $event)"
      @touchend="$emit('openPanel', $event)" @touchstart="$emit('startDrag', $event)"
    :cx='x' :cy='y' r="10" :fill="(showNode?color:'rgb(0,0,0,0)')" />
    <text v-if="showName" :x='x' :y='y+(showContent?115:20)' fill="white">{{name}}</text>
  </g>
</template>

<style scoped>
  text {
    user-select: none;
    text-anchor: middle;
    font-size: 12px;
  }
  .node circle:hover {
    transform: scale(1.2);
    transform-origin: center;
    transform-box: fill-box;
  }
</style>
