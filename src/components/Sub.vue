<script lang="ts">
import axios from 'axios';
import { ref } from "vue";
export default {
  components: { axios },
  setup() {
    const items: any = ref([])
    const getData = async (): Promise<Array<any>> => {
      const uri = window.location.search.substring(1); 
      const params = new URLSearchParams(uri);

      const authorization = params.get("authorization")
      const projectId = params.get("projectId")
      const taskId = params.get("taskId")

      const response = await axios.post(
        'https://studio.treed.ai/api/dashboard/status',
        {
          'id': taskId,
          'type': 'WORK',
          'keyword': '',
          'page': 0,
          'limit': 999999999,
          'order': 'asc',
          'sort': '',
          'filter': {
            'workStatuses': [
              'WORK_BEFORE'
            ],
            'operators': [
              'UNALLOCATED'
            ],
            'statuses': [
              'WORK_BEFORE'
            ]
          }
        },
        {
          headers: {
            'accept': 'application/json, text/plain, */*',
            'accept-language': 'en-US,en;q=0.9,vi-VN;q=0.8,vi;q=0.7,fr-FR;q=0.6,fr;q=0.5',
            'access-control-allow-origin': '*',
            'access-control-expose-headers': 'Authorization,Content-Disposition,Content-Type',
            'authorization': `${authorization}`,
            'content-type': 'application/json',
            'cookie': 'modalType=update; modalClosed=false;',
            'dnt': '1',
            'origin': 'https://studio.treed.ai',
            'priority': 'u=1, i',
            'referer': `https://studio.treed.ai/status?projectId=${projectId}&taskId=${taskId}`,
            'sec-ch-ua': '"Chromium";v="128", "Not;A=Brand";v="24", "Google Chrome";v="128"',
            'sec-ch-ua-mobile': '?0',
            'sec-ch-ua-platform': '"macOS"',
            'sec-fetch-dest': 'empty',
            'sec-fetch-mode': 'cors',
            'sec-fetch-site': 'same-origin',
            'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36'
          }
        }
      );
      return response.data.tasks
    }
    const getImgs = async (): Promise<Array<any>> => {
      const uri = window.location.search.substring(1); 
      const params = new URLSearchParams(uri);

      const authorization = params.get("authorization")
      const projectId = params.get("projectId")
      const taskId = params.get("taskId")

      const response = await axios.get('https://studio.treed.ai/api/workspace/items', {
        params: {
          'projectId': `${projectId}`,
          'taskId': `${taskId}`,
          'workType': 'MANAGER',
          'keyword': '',
          'page': '0',
          'limit': '999999999',
          'statuses': 'WORK_BEFORE',
          'operatorIds': ''
        },
        headers: {
          'accept': 'application/json, text/plain, */*',
          'accept-language': 'en-US,en;q=0.9,vi-VN;q=0.8,vi;q=0.7,fr-FR;q=0.6,fr;q=0.5',
          'access-control-allow-origin': '*',
          'access-control-expose-headers': 'Authorization,Content-Disposition,Content-Type',
          'authorization': `${authorization}`,
          'content-type': 'application/json',
          'cookie': `modalType=update; modalClosed=false;`,
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
      })
      return response.data.items
    }
    
    const getAllTasks = async () => {
      let allData: Array<any> = await getData()
      let allImg: Array<any> = await getImgs()
      let setNum = '0'
      for (const data of allData) {
        if (setNum != data.set) {
          data.isFirst = true
          setNum = data.set
          const img = allImg.find((e) => e.itemId === data.id)
          if (img) {
            data.filePath = img.filePath
          }
        }
      }
      items.value = allData
    }
    const getChange = async () => {
      const allData: Array<any> = await getData()
      const uniqNew = [...new Set(allData.map((e) => e?.set))]

      for (const item of items.value) {
        const idx = uniqNew.find((e) => e === item.set)
        if (idx === -1) {
          item.isFirst = false
        }
      }
    }
    return {
      items,
      getData,
      getImgs,
      getAllTasks,
      getChange,
    }
  },
  beforeMount() {
    this.getAllTasks()
  
    setInterval(()=> {
      this.getChange()
		}, 5000)
},
};
</script>
<template>
  <div class="m-4">
    <div class="grid grid-cols-4 gap-2">
      <template v-for="(item, index) in items" :key="index">
        <div v-if="item.isFirst">
          <label>{{ item.set }} {{ item.operatorName }}</label>
          <img :src="`https://treed-data-stable.s3.ap-northeast-2.amazonaws.com${item.filePath}`" width="500" height="500" >
        </div>
      </template>
    </div>
  </div>
</template>
