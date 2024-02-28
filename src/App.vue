<script setup>
import {ref,onBeforeMount,watch} from 'vue'
import Color from 'color'
import {ElColorPicker} from 'element-plus'

function blendColors(bottomColor, topColor) {
  const bottom = bottomColor.object();
  const top = topColor.object();
  let mix = calcAlpha(bottom.alpha,top.alpha)
  return Color([
    calcRGB(bottom.alpha,top.alpha,bottom.r,top.r),
    calcRGB(bottom.alpha,top.alpha,bottom.g,top.g),
    calcRGB(bottom.alpha,top.alpha,bottom.b,top.b)
  ]).alpha(mix)

}
function calcRGB(ba, ta, bc, tc) {
  let mix = calcAlpha(ba,ta)
  return Math.round((tc * (ta||1) / mix) + (bc * (ba||1) * (1 - (ta||1)) / mix));
}
function calcAlpha(ba,ta){
  return 1-(1-(ta||1))*(1-(ba||1))
}

function rgbaToArray(value){
  const matchs = value.replace(/\s/g, '').match(/^rgba?\((\d+),(\d+),(\d+),?([^,\s)]+)?/i)
  let rgbaArray = null
  if(matchs){
    let _matchs = [...matchs]
    _matchs.shift()
    rgbaArray =  _matchs.map(c=>{
      return parseFloat(c)
    })
  }
  return rgbaArray
}

const topColor = ref('rgba(0,255,0,0.3)')
const bottomColor = ref('rgba(255,0,0,0.5)')
const blendedColor =ref(null)
const mergeColor = ()=>{
  const bottomColorArray = rgbaToArray(bottomColor.value)
  const topColorArray = rgbaToArray(topColor.value)
  if(topColorArray && bottomColorArray){
    const topColor = Color(topColorArray).alpha(topColorArray[3])
    const bottomColor = Color(bottomColorArray).alpha(bottomColorArray[3])
    blendedColor.value = blendColors(bottomColor, topColor);
  }
}

onBeforeMount(()=>{
  mergeColor()
})
watch(bottomColor,mergeColor)
watch(topColor,mergeColor)

</script>

<template>
  <div class="color-merge">
    <div class="color-item color-bottom" @click="">
      <ElColorPicker v-model="bottomColor" show-alpha />
      <div>背景色</div>
      <div>{{bottomColor}}</div>
    </div>
    <div class="color-item color-top" @click="">
      <ElColorPicker v-model="topColor" show-alpha />
      <div>前景色</div>
      <div>{{topColor}}</div>
    </div>
  </div>
  <div class="merge-res">
    <div class="merge-res-con">
    </div>
    <div>
      {{blendedColor.toString()}}
    </div>
    <div style="width: 1px;height: 18px;border-right: 1px solid #ddd"></div>
    <div>
      {{blendedColor.hexa()}}
    </div>
  </div>
</template>

<style scoped>
.color-merge {
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}
.color-item {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 400px;
  height: 400px;
  border-radius: 50%;
  z-index: 2;
  text-align: center;
  flex-flow: column;
  grid-gap: 4px;
}
.color-top{
  margin-left: -50px;
  background-color:v-bind(topColor)
}
.color-bottom{
  margin-right: -50px;
  background-color:v-bind(bottomColor)
}
.merge-res{
  border: 1px solid #ebebeb;
  padding: 4px;
  margin-top: 48px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  grid-gap: 4px;
}
.merge-res-con{
  width: 32px;
  height: 32px;
  background-color: v-bind(blendedColor);
  z-index: 2;
}
.background-text{
  position: absolute;
  z-index: 1;
  left: 0;
  right: 0;top: 0;
  bottom: 0;
  display: flex;
  align-items: center;
  justify-content: center
}
</style>
