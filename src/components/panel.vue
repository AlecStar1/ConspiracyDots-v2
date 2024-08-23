<script setup>
import { ref, onMounted} from 'vue'
const props = defineProps({
  node:{
    type: Object
  },
  startX:{
    type:Number
  },
  startY:{
    type:Number
  },
  showName:{
    type:Boolean
  }
})
const x = ref(props.startX) 
const y = ref(props.startY)
const rand = Math.random()
var down = false
const emit = defineEmits(['content'])
function math(e) {
  return [e.clientX - document.getElementById('out').offsetWidth/2, e.clientY - document.getElementById('out').offsetHeight/8];
}
function r(){
  if(document.getElementById(rand+"name"))
  {var textarea = document.getElementById(rand+"name");
var heightLimit = 100; /* Maximum height: 200px */
  textarea.style.height = "10px"; /* Reset the height*/
  textarea.style.height = Math.min(textarea.scrollHeight, heightLimit) + "px";}
}
onMounted(r)
const defaultImage = "https://www.mediacollege.com/adobe/premiere/version/pro-cs6/images/CS6_media-offline.png"
function encodeImageFileAsURL(element) {
  var file = element.files[0];
  var reader = new FileReader();
  
  reader.onloadend = function() {
    emit('content', reader.result)
  }
  reader.readAsDataURL(file);
}
</script>
<template>
  <div v-if="node != undefined" id="out" class="outer" :style="'transform:translate('+x+'px,'+y+'px)'" 
  @mouseup="down = false" @mousemove="(e) => {if(down){[x, y] = math(e)}}">
        <span @mousedown="down = true"></span>
    <a href="#" class="closeButton" @click="$emit('destroyWindow', $.vnode.key)">X</a>
    <div class="header"><form class="header">
      <textarea maxlength="20" :id="rand+'name'" cols="20" wrap="hard" class="name" type="text" :placeholder="node.name" @input="r" @change="($event.target.value != '')?$emit('name', $event):''"/>
      <input type="color" :value="node.color" @change="$emit('color', $event)"/></form>
      <div class="show">
        <input type="checkbox" :id="rand+'node'" :checked="node['showNode'] != null?node['showNode']:true" @change="$emit('showNode', $event.target.checked)" />
        <input type="checkbox" :id="rand+'name'" :checked="node['showName']" @change="$emit('showName', $event.target.checked)" />
        <input type="checkbox" :disabled="node['type'] == ''" :id="rand+'content'" :checked="node['showContent']  != null?node['showContent']:node['type']=='image'" @change="$emit('showContent', $event.target.checked)" />
      </div>
    </div>
    <img v-if="node['type'] === 'image'" :src="!node['content']||node['content']==''?defaultImage:node['content']" width="150vw" height="150vw"/>
    <input type="file" v-if="node['type'] === 'image'" @change="encodeImageFileAsURL($event.target)"/>
    <textarea v-else-if="node['type'] === 'text'" :value="node.content" placeholder="Not Provided." @change="$emit('content', $event.target.value)"/>
    <select v-else @change="$emit('pickType', $event.target.value)">
      <option value="">Pick a type:</option>
      <option value="text">Text</option>
      <option value="image">Image</option>
    </select>
  </div>
</template>

<style scoped>
  img {
    align-self: center;
  }
  span {
    right: 20%;
    left: 20%;
    top: 0;
    height: 10px;
    position: absolute;
    cursor: grab;
  }
  .outer {
    display: flex;
    flex-direction: column;
    row-gap: 1vw;
    position: fixed;
    background-color: rgb(161, 57, 57);
    padding: 1%;
    border-radius: 10px;
    width: 20vw;
  }
  a {
    user-select: none;
  }
  .closeButton{
    position: absolute;
  }
  textarea.name{
    width: 10vw;
    text-align: center;
    color: black;
    background-color: rgb(0,0,0,0);
    border: none;
    font-family: serif;
    font-weight: bold;
    font-size: 2.25vw;
    box-sizing: border-box;
    resize: none;
    height: 10%;
  }
  textarea.name::placeholder {
    color: black;
  }

  input[type="color"]:first-child {
    padding: 0;
    margin: 0;
    border: none;
    box-shadow: none;
    border-radius: 100px;
    background: none;
    margin-bottom: 20px;
    flex: 1;
  }

  input[type="color"]::-webkit-color-swatch-wrapper {
    padding: 0;
  }
  input[type="file"]{
    appearance:initial;
    color: transparent;
  }

 

  input[type="color"]::-webkit-color-swatch {
    border: none;
    border-radius: 400px;
  }

  input[type="color"]:nth-child(2) {
    padding: 0;
    margin: 0;
    border: none;
    box-shadow: none;
    border-radius: 1000px;
    background: #f0bc12;
    outline: none;
  }

  .show {
    position: fixed;
    display: flex;
    flex-direction: column;
    right: 1%;
    top: 1%;
  }

  input[type="color" i] {
    border-radius: 400px;
    border: none;
    height: 5vw;
    width: 5vw;
    display: block;
  }
  .header{
    display: flex;
    flex-direction: row;
    width: 20vw;
    justify-content:space-around;
    align-items:center;
    
  }

</style>
