<script setup>
/*
* importing
*/

import { ref, reactive, computed, watch } from 'vue'
import node from './components/node.vue'
import panel from './components/panel.vue'

/*
* Variables
*/ 
const nodes = ref({})
const lines = ref({})
const open = ref({})
let newNoded = false
let currentDrag = null
let movement = false
let originalPos = ref([])
const panelsOpen = ref({})
const m = ref([0, 0])
const hovering = ref('')
var socket = io();



socket.on('updateNode', function(node){
  nodes.value[node.id] = node
})
socket.on('deleteNode', function(nodeID){
  delete nodes.value[nodeID]
})
socket.on('linkNode', function(nodeID){
  lines.value = nodeID
})

/*
 * Functions
*/

const linesFiltered = computed(() => {
  var out = lines.value
  for (const [i, f] of Object.entries(out)) {
    for (const [j, value] of Object.entries(f)) {
      if(!out[i][j])delete out[i][j]
    }
  }
  return out
})

function newNode(x, y, id = Math.random()) {
  if (newNoded) {
    nodes.value[id] = {
      x: x,
      y: y,
      name: "foo",
      color: "#F0F0F0",
      type:'',
      showName: true,
      id: id
    }
    // socket.emit('nodeUpdated',  nodes.value[id])
    socket.emit('nodeUpdated', nodes.value[id])
    watch(nodes.value[id],
    (e) => {
      console.log(e)
      socket.emit('nodeUpdated', e)
    }
    )
    newNoded = false
  }
}

function getX(percent) {
  return percent * window.innerWidth
}
function getY(percent) {
  return percent * window.innerHeight
}
function toPercentX(X){
  return X / window.innerWidth
}
function toPercentY(X){
  return X / window.innerHeight
}

function findStateOfLink(id1, id2){
  return (
    (((lines.value[id1] || {})[id2]))?1:
    ((((lines.value[id2] || {})[id1]))?2:0)
  );
}

function panelList(dom, x, y) {
  this.dom = dom
  this.x = x
  this.y = y
}
function pointInCircle(x, y, cx, cy, radius) {
  var distancesquared = (x - cx) * (x - cx) + (y - cy) * (y - cy);
  return distancesquared <= radius * radius;
}
function getNodesUnderPosition(x, y) {
    var d = [];
    for (const [key, i] of Object.entries(nodes.value))
      if(pointInCircle(x,y,getX(i.x),getY(i.y),10)) 
        d.push(key)
    return d;
}

/*
 * Event Listeners
*/

window.addEventListener("mousemove", function (e) {
  const x = e.clientX / window.innerWidth
  const y = e.clientY / window.innerHeight
  if (currentDrag) {
    nodes.value[currentDrag].x = x
    nodes.value[currentDrag].y = y
    let under = getNodesUnderPosition(e.clientX, e.clientY);
    under.splice(under.indexOf(currentDrag),1)
    hovering.value = (under.length!=0)?under[0]:""
    movement = true
  }
})

window.addEventListener("mouseup", function (e) {
  if(currentDrag){
    let nodesUnderCursor = getNodesUnderPosition(e.clientX, e.clientY)
    hovering.value = ""
    if(nodesUnderCursor.length > 1){
      [nodes.value[currentDrag].x, nodes.value[currentDrag].y] = originalPos.value
      switch(findStateOfLink(nodesUnderCursor[0], nodesUnderCursor[1])){
        case 0:
          if(!lines.value[nodesUnderCursor[0]]) lines.value[nodesUnderCursor[0]] = {}
          lines.value[nodesUnderCursor[0]][nodesUnderCursor[1]] = true
          
          break;
        case 1:
          lines.value[nodesUnderCursor[0]][nodesUnderCursor[1]] = false
          // socket.emit('linkChange', lines)
          break;
        case 2:
          lines.value[nodesUnderCursor[1]][nodesUnderCursor[0]] = false
          // socket.emit('linkChange', lines)
          break;
        default:
          throw new Error("Unexpected value.")
      }
      socket.emit('linkChange', lines.value)
    } 
  }
  currentDrag = null
})

window.addEventListener("touchmove", function (e) {
  const x = e.touches[0].clientX  / window.innerWidth
  const y = e.touches[0].clientY / window.innerHeight
  
  if (currentDrag) {
    nodes.value[currentDrag].x = x
    nodes.value[currentDrag].y = y
    let under = getNodesUnderPosition(e.touches[0].clientX, e.touches[0].clientY);
    under.splice(under.indexOf(currentDrag),1)
    hovering.value = (under.length!=0)?under[0]:""
    movement = true
  }
})

window.addEventListener("touchend", function (e) {
  if(currentDrag){
    let nodesUnderCursor = getNodesUnderPosition(e.changedTouches[0].clientX, e.changedTouches[0].clientY)
    hovering.value = ""
    if(nodesUnderCursor.length > 1){
      [nodes.value[currentDrag].x, nodes.value[currentDrag].y] = originalPos.value
      switch(findStateOfLink(nodesUnderCursor[0], nodesUnderCursor[1])){
        case 0:
          if(!lines.value[nodesUnderCursor[0]]) lines.value[nodesUnderCursor[0]] = {}
          lines.value[nodesUnderCursor[0]][nodesUnderCursor[1]] = true
          break;
        case 1:
          lines.value[nodesUnderCursor[0]][nodesUnderCursor[1]] = false
          break;
        case 2:
          lines.value[nodesUnderCursor[1]][nodesUnderCursor[0]] = false
          break;
        default:
          throw new Error("Unexpected value.")
      }
      
    }
  }
  currentDrag = null
})

</script>

<template>

  
  <button class="addButton" :disabled="newNoded" @click="newNoded = true">+</button>
  <panel @destroyWindow="(f) => { open['' + f] = undefined; panelsOpen[i] = undefined }" 
         @color="(e) => {nodes[i]['color'] = e.target.value}"
         @name="(e) => {nodes[i]['name'] = e.target.value}"
         @content="(e) => {nodes[i]['content'] = e}"
         @showName="(e) => {nodes[i]['showName'] = e}"
         @showNode="(e) => {nodes[i]['showNode'] = e}"
         @showContent="(e) => {nodes[i]['showContent'] = e}"
         @pickType="(e) => {if(e == 'image') nodes[i].showContent = true;nodes[i]['type'] = e}" 
         :ref="(e) => { panelsOpen[i] = new panelList(e, m[0], m[1]) }" 
         v-for="i in open" 
         :node="nodes[i]" 
         :key="i" 
         :startX="m[0]" 
         :startY="m[1]"
         />
  <svg @click="newNoded ? newNode(toPercentX($event.clientX), toPercentY($event.clientY)) : ''" class="wrapper">
    
    <g v-for="j,k in linesFiltered" class="link">
      <path v-for="i,h in j"
          stroke-width="1" 
          stroke="lightgray" 
          :d="'M' + getX(nodes[k].x) + ',' + getY(nodes[k].y) + ',' + getX(nodes[h].x) + ',' + getY(nodes[h].y)"
          />
      </g>
    <circle
      class="selector"
      fill="rgb(0,0,0,0)"
      stroke="lightgray"
      stroke-width="2"
      r="15"
      v-if="hovering != ''"
      :cx="getX(nodes[hovering].x)"
      :cy="getY(nodes[hovering].y)"
    />
    <node v-for="(item, key) in nodes" 
          @openPanel="((!movement) ? ((!open[key]) ? open[key] = key : open[key] = undefined) : movement = false); m = [$event.clientX, $event.clientY]" 
          @startDrag="currentDrag = key; originalPos = [item.x, item.y]"
           
          :key="key" 
          v-bind="item" 
          :x="getX(item.x)" 
          :y="getY(item.y)" />
  </svg>
</template>

<style scoped>
svg, body {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0 !important;
  background-color: #252525;
  
}
.addButton {
  position: absolute;
  left: 1%;
  bottom: 1%;
  width: 10vw;
  height: 10vw;
  font-size: 8vw;
}
</style>

