<script lang="ts">
import Moveable from "vue3-moveable";
import { ref } from "vue";
export default {
  components: { Moveable },
  setup() {
    const picturesRef: any = ref([]);
    const currentIndex = ref(undefined);
    const mouseEnter = ref(false);
    const draggable = true;
    const resizable = true;
    const onDelete = (index: number) => {
      picturesRef.value.splice(index, 1)
    }
    const onDrag = (e: any) => {
      e.target.style.transform = e.transform;
    };
    const onResize = (e: any) => {
      e.target.style.width = `${e.width}px`;
      e.target.style.height = `${e.height}px`;
      e.target.style.transform = e.drag.transform;
    };
    const onMouseOut = () => {
      if (!mouseEnter.value) {
        currentIndex.value = undefined
      }
      picturesRef.value.sort((a, b) => a.seq - b.seq)
      for (let i = 0; i < picturesRef.value.length; i++) {
        picturesRef.value[i].seq = i + 1
      }
    };
    const pasteFunction = (pasteEvent: any, callback: any) => {
      if (pasteEvent.clipboardData == false) {
        if (typeof callback == "function") {
          console.log("Undefined ");
          callback(undefined);
        }
      }
      var items = pasteEvent.clipboardData.items;
      if (items == undefined) {
        if (typeof callback == "function") {
          callback(undefined);
          console.log("Undefined 2");
        }
      }
      for (var i = 0; i < items.length; i++) {
        if (items[i].type.indexOf("image") == -1) continue;
        var blob = items[i].getAsFile();
        const reader = new FileReader()
        reader.readAsDataURL(blob)
        reader.onload = ((e) => {
          if (e.target && e.target.result) {
            picturesRef.value.push({ seq: picturesRef.value.length + 1, img: e.target.result.toString() })
          }
        })
      }
    };
    return {
      picturesRef,
      currentIndex,
      mouseEnter,
      draggable,
      resizable,
      onDelete,
      onDrag,
      onResize,
      onMouseOut,
      pasteFunction,
    };
  },
};
</script>
<template>
  <div
    class="flex flex-col px-2 py-2"
    @click.prevent="onMouseOut"
    @paste.prevent="pasteFunction"
  >
    <div v-if="picturesRef.length === 0" class="text-blue-500 text-4xl">Please command/ctrl V to paste img</div>
    <div v-for="(picture, index) in picturesRef" :key="index" class="relative">
      <div class="absolute z-40" v-if="currentIndex === index">
        <div class="flex flex-row" @mouseenter="mouseEnter = true" @mouseleave="mouseEnter = false">
          <div class="" @click.prevent="onDelete(index)">
            <svg height="30px" width="30px" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <path d="M10 12V17" stroke="#ff0000" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path> <path d="M14 12V17" stroke="#ff0000" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path> <path d="M4 7H20" stroke="#ff0000" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path> <path d="M6 10V18C6 19.6569 7.34315 21 9 21H15C16.6569 21 18 19.6569 18 18V10" stroke="#ff0000" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path> <path d="M9 5C9 3.89543 9.89543 3 11 3H13C14.1046 3 15 3.89543 15 5V7H9V5Z" stroke="#ff0000" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path> </g></svg>
          </div>
          <div class="ml-4 text-cyan-500">
            <input class="w-20" v-model="picture.seq">
          </div>
        </div>
      </div>
      <img
        @click.prevent="currentIndex = index"
        @mouseenter="mouseEnter = true"
        @mouseleave="mouseEnter = false"
        class="border border-slate-300 z-0"
        :class="{ target: currentIndex === index }"
        :src="picture.img"
      />
      <div v-if="index + 1 !== picturesRef.length" class="flex items-center justify-center">
      <svg height="70px" width="60px" version="1.1" id="Capa_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="-0.96 -0.96 33.92 33.92" xml:space="preserve" fill="#ff0000" stroke="#ff0000" stroke-width="0.608"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <g> <g id="arrow_x5F_down"> <path style="fill:#ff0000;" d="M32,16.016l-5.672-5.664c0,0-3.18,3.18-6.312,6.312V0h-8.023v16.664l-6.32-6.32L0,16.016L16,32 L32,16.016z"></path> </g> </g> </g></svg>
      </div>
      <Moveable
        v-if="currentIndex === index"
        :target="['.target']"
        :draggable="draggable"
        :resizable="resizable"
        @drag="onDrag"
        @resize="onResize"
      />
    </div>
  </div>
</template>
