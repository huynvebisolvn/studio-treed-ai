<script lang="ts">
import axios from 'axios';
import { ref } from "vue";
export default {
  components: { axios },
  setup() {
    const items: any = ref([])
    const childItems: any = ref([])
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
    const deleteRequest = async (itemId: number) => {
      const uri = window.location.search.substring(1); 
      const params = new URLSearchParams(uri);

      const authorization = params.get("authorization")
      const taskId = params.get("taskId")

      const response = await axios.post(
        `https://studio.treed.ai/api/workspace/item/${itemId}/execute/delete`,
        {
          'workType': 'OPERATOR'
        },
        {
          headers: {
            'accept-language': 'en-US,en;q=0.9,vi-VN;q=0.8,vi;q=0.7,fr-FR;q=0.6,fr;q=0.5',
            'access-control-allow-origin': '*',
            'access-control-expose-headers': 'Authorization,Content-Disposition,Content-Type',
            'authorization': `${authorization}`,
            'cookie': 'modalType=update; modalClosed=false;',
            'dnt': '1',
            'origin': 'https://studio.treed.ai',
            'priority': 'u=1, i',
            'referer': `https://studio.treed.ai/workspace/${taskId}?workType=OPERATOR`,
            'sec-ch-ua': '"Chromium";v="128", "Not;A=Brand";v="24", "Google Chrome";v="128"',
            'sec-ch-ua-mobile': '?0',
            'sec-ch-ua-platform': '"Windows"',
            'sec-fetch-dest': 'empty',
            'sec-fetch-mode': 'cors',
            'sec-fetch-site': 'same-origin',
            'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36'
          }
        }
      );
    }
    const getPics = async () => {
      const allData: Array<any> = await getData()

      // count pic on a set
      const countSetNum = allData.filter((e) => e.setNum === allData.slice(-1).pop()?.setNum).length

      // check item not a set (10)
      let setNum = 0
      for (const data of allData) {
        if (setNum != data.setNum) {
          data.isShow = true
          setNum = data.setNum
        }
        const countCurrent = allData.filter((e) => e.setNum === data.setNum).length
        if (countCurrent != countSetNum) {
          data.isShow = false
        }
      }
      
      // control not change status
      const allIsShow = allData.filter((e) => e.isShow)
      const uniq = [...new Set(allIsShow.map((e) => e?.setNum))]
      let after = []
      for (let i = uniq.length - 1 ; i >= 0; i--) {
          if (uniq[i] !== uniq[i-1] + 1) {
              after.push(uniq[i])
              break
          }
          after.push(uniq[i])
      }

      for (const data of allData) {
        if (data.isShow) {
          const idx = after.findIndex((e) => e === data.setNum)
          if (idx === -1) {
            data.isShow = false
          }
        }
      }

      return allData
    }
    const setData = async () => {
      const allPics: Array<any> = await getPics()
      items.value = allPics
    }
    const getChange = async () => {
      const allPics: Array<any> = await getPics()
      const allPicsShow = allPics.filter((e) => e.isShow)
      const uniq = [...new Set(allPicsShow.map((e) => e?.setNum))]

      // find new id to remove
      for (const item of items.value) {
        const doneIdx = uniq.findIndex((e) => e === item.setNum)
        if (doneIdx === -1) {
          item.isShow = false
        }
      }
    }
    const getChildItem = async (_setNum: number) => {
      childItems.value = []
      childItems.value = items.value.filter((e) => e.setNum === _setNum)
    }
    const clearChildItem = async () => {
      childItems.value = []
    }
    const hasTask = async (_setNum: Number) => {
      if (confirm('Bấm OK hoặc bấm ENTER để lấy set ' + _setNum + '?')) {
        const listItems = items.value.filter((e) => e.setNum === _setNum)
        for (const item of listItems) {
          await deleteRequest(item.itemId)
        }
      }
    }
    return {
      items,
      childItems,
      getData,
      deleteRequest,
      getPics,
      setData,
      getChange,
      getChildItem,
      clearChildItem,
      hasTask,
    }
  },
  beforeMount() {
    this.setData()
  
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
        <div v-if="item.isShow">
          <button
            type="button"
            class="text-white bg-blue-700 hover:bg-blue-800 focus:outline-none font-medium rounded-lg px-1 text-center dark:bg-blue-600 dark:hover:bg-blue-700"
            @click="hasTask(item.setNum)"
          >
            Lấy Hết {{ item.setNum }}
          </button>
          <img :src="`https://treed-data-stable.s3.ap-northeast-2.amazonaws.com${item.filePath}`" @click="getChildItem(item.setNum)" >
        </div>
      </template>
    </div>
    <div v-if="childItems.length > 0" tabindex="-1" aria-hidden="true" class="fixed top-[5%] right-0 left-[10%] z-50 justify-center w-[80%] h-[80%]">
      <div class="relative p-4 w-full h-full">
        <div class="relative rounded-lg shadow border-4 bg-gray-300">
          <div class="grid grid-cols-4 gap-2 m-4">
            <template v-for="(childitem, childindex) in childItems" :key="childindex">
              <div>
                <label>{{ childitem.setNum }} - {{ childindex + 1 }} - {{ childitem.itemId }}</label>
                <img :key="childindex" :src="`https://treed-data-stable.s3.ap-northeast-2.amazonaws.com${childitem.filePath}`" >
              </div>
            </template>
          </div>
          <div class="flex items-center p-2 md:p-5 rounded-b dark:border-gray-600">
              <button data-modal-hide="default-modal" type="button" class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 text-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800" @click="hasTask(childItems[0].setNum)">Lấy Hết</button>
              <button data-modal-hide="default-modal" type="button" class="py-2.5 px-5 ms-3 text-sm font-medium text-gray-900 focus:outline-none bg-white rounded-lg border border-gray-200 hover:bg-gray-100 hover:text-blue-700 focus:z-10 focus:ring-4 focus:ring-gray-100 dark:focus:ring-gray-700 dark:bg-gray-800 dark:text-gray-400 dark:border-gray-600 dark:hover:text-white dark:hover:bg-gray-700" @click="clearChildItem()">Bỏ Qua</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
