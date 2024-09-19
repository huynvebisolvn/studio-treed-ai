<script lang="ts">
import axios from 'axios';
import { ref } from "vue";
export default {
  components: { axios },
  setup() {
    const items: any = ref([]);
    const getPics = async () => {
      const authorization = '506ca703eec4e698bbc8d8a9a366244c8d1db552247811cc5d882a522a5b620f242844b4cf99ce48c915e9e272c5cf9bd25143b5afcd190d100016f73ceaa2989cda1ead76b7123dd70697cec93e03b7';
      const jsessionid = 'BEFA85780CE88A963C1B9BDAAF3FA212'
      const projectId = '221'
      const taskId = '2179'

      const response = await axios.get('https://studio.treed.ai/api/workspace/items', {
        params: {
          'projectId': `${projectId}`,
          'taskId': `${taskId}`,
          'workType': 'MANAGER',
          'keyword': '',
          'page': '0',
          'limit': '30',
          'statuses': '',
          'operatorIds': ''
        },
        headers: {
          'accept': 'application/json, text/plain, */*',
          'accept-language': 'en-US,en;q=0.9,vi-VN;q=0.8,vi;q=0.7,fr-FR;q=0.6,fr;q=0.5',
          'access-control-allow-origin': '*',
          'access-control-expose-headers': 'Authorization,Content-Disposition,Content-Type',
          'authorization': `${authorization}`,
          'content-type': 'application/json',
          'cookie': `modalType=update; modalClosed=false; JSESSIONID=${jsessionid}`,
          'dnt': '1',
          'priority': 'u=1, i',
          'referer': `https://studio.treed.ai/workspace/${taskId}?workType=MANAGER`,
          'sec-ch-ua': '"Chromium";v="128", "Not;A=Brand";v="24", "Google Chrome";v="128"',
          'sec-ch-ua-mobile': '?0',
          'sec-ch-ua-platform': '"macOS"',
          'sec-fetch-dest': 'empty',
          'sec-fetch-mode': 'cors',
          'sec-fetch-site': 'same-origin',
          'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36'
        }
      });
      const allData = response.data.items
      let setNum = 0;
      for (const data of allData) {
        if (setNum != data.setNum) {
          items.value.push(data)
          setNum = data.setNum
        }
      }
    }
    return {
      items,
      getPics,
    };
  },
  beforeMount() {
    this.getPics()
},
};
</script>
<template>
  <div>
    <div class="grid grid-cols-4 gap-2">
      <div v-for="(item, index) in items" :key="index">
        {{ item.setNum }}
        <img :src="`https://treed-data-stable.s3.ap-northeast-2.amazonaws.com${item.filePath}`" width="500" height="500" >
      </div>
    </div>
  </div>
</template>
