<script setup lang="ts">
import axios, { type AxiosResponse, type AxiosRequestConfig, type RawAxiosRequestHeaders} from 'axios';
import Sidebar from "./Sidebar.vue";

defineProps<{
  msg: string
}>()
let data = JSON.parse(localStorage.getItem("autosave"));

if(data) {
  console.log('localStorage data = '+data); 
}
else {
  const client = axios.create({
    baseURL: 'https://64b4c8450efb99d862694609.mockapi.io',
  });
  const config: AxiosRequestConfig = {
      headers: {
        'Accept': 'application/json',
      } as RawAxiosRequestHeaders,
    };

  const response: AxiosResponse = await client.get('/tree/items', config);

  const titles = response.data;
  const parent = {parent:true};

  let ids = [];
  let parent_ids = [];
  for(let i = 0; i < titles.length; i++){
    ids.push(Number.parseInt(titles[i].id));
    parent_ids.push(Number.parseInt(titles[i].parent_id==null?0:titles[i].parent_id));
  }

  let max = Math.max(...ids);
  let maxParent = Math.max(...parent_ids);

  max = Math.max(max, maxParent);

  const root = Map.groupBy(titles, ({ parent_id }) =>
      parent_id == null ? parent : null
  );

  var result = new Map();
  var count = 0;
  while(count!=max) {
    count++;
    result = Map.groupBy(root.get(null), ({parent_id}) => 
      parent_id == count ? count : null
    );
    let temp = result.get(count) !== undefined?root.set(count, result.get(count)):null;
    root.set(null, result.get(null));
  }

  let searchParent = (data, index, level) => {
    while(data.length>index){
      if(root.get(Number.parseInt(data[index].id))!=undefined){
        data[index]["items"] = root.get(Number.parseInt(data[index].id));
        data[index]["toggled"] = false;
        data[index]["level"] = level;
        searchParent(data[index]["items"],index,level+1);
      }
      index++;
    }
  }

  data = root.get(parent);
  searchParent(data,0, 0);
  console.log(data);

  localStorage.setItem("autosave", JSON.stringify(data));
}

// [{ name: "bananas", type: "fruit", quantity: 5 }]
</script>

  <template>
    <div class="greetings">
      <h1 class="green">{{ msg }}</h1>
      <h3>
        You’ve successfully created a project with
        <a href="https://vitejs.dev/" target="_blank" rel="noopener">Vite</a> +
        <a href="https://vuejs.org/" target="_blank" rel="noopener">Vue 3</a>.
      </h3>
      <Sidebar :level="1" :items="data" />
    </div>
  </template>

<style scoped>

h1 {
  font-weight: 500;
  font-size: 2.6rem;
  position: relative;
  top: -10px;
}

h3 {
  font-size: 1.2rem;
}

.greetings h1,
.greetings h3 {
  text-align: center;
}

@media (min-width: 1024px) {
  .greetings h1,
  .greetings h3 {
    text-align: left;
  }
}
</style>
