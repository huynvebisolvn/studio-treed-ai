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
    const getPics = async () => {
      items.value = []
     
      const allData: Array<any> = await getData()
      const doneList: Array<any> = allData.filter((e) => e?.operatedDate)
      const uniq = [...new Set(doneList.map((e) => e?.setNum))]

      // get first item as new set
      let setNum = 0
      let tempItem = []
      for (const data of allData) {
        if (setNum != data.setNum) {
          const doneIdx = uniq.findIndex((e) => e === data.setNum)
          if (doneIdx === -1) {
            tempItem.push(data)
            setNum = data.setNum
          }
        }
      }

      // control not change status
      let after = []
      for (let i = tempItem.length - 1 ; i >= 0; i--) {
          if (tempItem[i]?.setNum !== tempItem[i-1]?.setNum + 1) {
              after.push(tempItem[i])
              break
          }
          after.push(tempItem[i])
      }

      items.value = after.reverse()
    }
    const getChange = async () => {
      const allData: Array<any> = await getData()
      const doneList: Array<any> = allData.filter((e) => e?.operatedDate)
      const uniq = [...new Set(doneList.map((e) => e?.setNum))]

      //  get first item as new set
      let setNum = 0
      let tempItem = []
      for (const data of allData) {
        if (setNum != data.setNum) {
          const doneIdx = uniq.findIndex((e) => e === data.setNum)
          if (doneIdx === -1) {
            tempItem.push(data)
            setNum = data.setNum
          }
        }
      }

      // find new id to remove
      let setNumRemove = []
      for (const item of items.value) {
        const doneIdx = tempItem.findIndex((e) => e?.setNum === item.setNum)
        if (doneIdx === -1) {
          setNumRemove.push(item.setNum)
        }
      }

      for (const rmNum of setNumRemove) {
        const removeIdx = items.value.findIndex((e) => e?.setNum === rmNum)
        items.value.splice(removeIdx, 1)
      }
    }
    return {
      items,
      getData,
      getPics,
      getChange,
    }
  },
  beforeMount() {
    this.getPics()
  
    setInterval(()=> {
      this.getChange()
		}, 5000)
},
};
</script>
<template>
  <div class="m-4">
    <div class="grid grid-cols-4 gap-2">
      <div v-for="(item, index) in items" :key="index">
        {{ item.setNum }} - {{ item.operatedDate }}
        <img :src="`https://treed-data-stable.s3.ap-northeast-2.amazonaws.com${item.filePath}`" width="500" height="500" >
      </div>
    </div>
  </div>
</template>
