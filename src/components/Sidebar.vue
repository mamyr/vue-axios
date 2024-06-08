<!-- eslint-disable vue/no-mutating-props -->
// eslint-disable-next-line vue/multi-word-component-names
<script setup lang="ts">
import RecursiveSidebar from "./Sidebar.vue";

type Item = {
    level: number;
    title: string;
    items: Item[];
};

const {items} = defineProps<{
  level: number;
  items: Item[];
}>();

let searchParent = (data, index, item) => {
    while(data.length>index && item.id!=data[index].id){
        if(data[index]["items"])
            searchParent(data[index]["items"],index,item);
        index++;
    }
    if(data[index]){
        data[index]["toggled"] = item.toggled;
        return;
    }    
}

//const search = (data, item) => data.find((d) => {if(d.id === item.id) d.toggled = item.toggled; return d;}) || search(d.items, item);
const changeToggled = (item) => {
    item.toggled = !item.toggled
    let data = JSON.parse(localStorage.getItem("autosave"));
    searchParent(data,0,item);
    localStorage.setItem("autosave", JSON.stringify(data));
}
    
</script>

<template>
  <ul class="card">
    <li v-for="(item,index) in items" :item="item" :index="index" :key="item.title"  
    :style="{backgroundColor: [level==1 ? '#b8b0b0' : ( level%2==0 ? (index%2==0 ? 'white' : '#e3d3d3'): (index%2==0 ? '#e3d3d3' : 'white') )]
    }">
      <details @click="changeToggled(item)" :open="item.toggled">
        <summary class="flex group my-1 text-left max-w-xs" v-if="item.items && item.items.length > 0">
            {{ item.title }}
        </summary>
        <summary class="flex group my-1 text-left max-w-xs markerS" v-if="!(item.items && item.items.length > 0)" style="{list-style-type: '-'; ::marker {display:none}}">
            {{ item.title }}
        </summary>
          <RecursiveSidebar
            v-if="item.items && item.items.length > 0"
            :items="item.items"
            :level="level + 1"
          />
      </details>
    </li>
  </ul>
</template>

<style scoped>
    ul {
        list-style-type: none;
    }
    .markerS {
        list-style-type: '-';
        summary::-webkit-details-marker, summary::marker {
            display: none;
        }
    }
    .backgroundC1 {
        background-color: "#b8b0b0";
    }
    .backgroundC2 {
        background-color: "white";
    }
    .backgroundC3 {
        background-color: "#e3d3d3";
    }
</style>