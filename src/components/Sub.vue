<script lang="ts">
import axios from 'axios';
import { ref } from "vue";
export default {
  components: { axios },
  setup() {
    const params = ref({ authorization: '', projectId: '', taskId: '' })
    const loading: any = ref(true)
    const users: any = ref([])
    const usersTask: any = ref([])
    const items: any = ref([])
    const wishList: any = ref([])
    const childItems: any = ref([])
    const funTimer = (ms: number) => new Promise((res) => setTimeout(res, ms))
    const funGetParams = () => {
      const uri = window.location.search.substring(1); 
      const urlParams = new URLSearchParams(uri);
      params.value.authorization = urlParams.get("authorization")
      params.value.projectId = urlParams.get("projectId")
      params.value.taskId = urlParams.get("taskId")
    }
    const funGetUsers = async () => {
      const response = await axios.get(`https://studio.treed.ai/api/dashboard/status/people/${params.value.taskId}/WORK`, {
        headers: {
          'accept': 'application/json, text/plain, */*',
          'accept-language': 'en-US,en;q=0.9,vi-VN;q=0.8,vi;q=0.7,fr-FR;q=0.6,fr;q=0.5',
          'access-control-allow-origin': '*',
          'access-control-expose-headers': 'Authorization,Content-Disposition,Content-Type',
          'authorization': `${params.value.authorization}`,
          'content-type': 'application/json',
          'cookie': 'modalType=update; modalClosed=false;',
          'dnt': '1',
          'priority': 'u=1, i',
          'referer': `https://studio.treed.ai/status?projectId=${params.value.projectId}&taskId=${params.value.taskId}`,
          'sec-ch-ua': '"Chromium";v="128", "Not;A=Brand";v="24", "Google Chrome";v="128"',
          'sec-ch-ua-mobile': '?0',
          'sec-ch-ua-platform': '"macOS"',
          'sec-fetch-dest': 'empty',
          'sec-fetch-mode': 'cors',
          'sec-fetch-site': 'same-origin',
          'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36'
        }
      });
      users.value = response.data.operators
    }
    const funGetTaskByUser = async (userId: string): Promise<Array<any>> => {
      const response = await axios.get('https://studio.treed.ai/api/workspace/items', {
        params: {
          'projectId': `${params.value.projectId}`,
          'taskId': `${params.value.taskId}`,
          'workType': 'MANAGER',
          'keyword': '',
          'page': '0',
          'limit': '999999999',
          'statuses': 'WORK_BEFORE',
          'operatorIds': `${userId}`
        },
        headers: {
          'accept': 'application/json, text/plain, */*',
          'accept-language': 'en-US,en;q=0.9,vi-VN;q=0.8,vi;q=0.7,fr-FR;q=0.6,fr;q=0.5',
          'access-control-allow-origin': '*',
          'access-control-expose-headers': 'Authorization,Content-Disposition,Content-Type',
          'authorization': `${params.value.authorization}`,
          'content-type': 'application/json',
          'cookie': `modalType=update; modalClosed=false;`,
          'dnt': '1',
          'priority': 'u=1, i',
          'referer': `https://studio.treed.ai/workspace/${params.value.taskId}?workType=MANAGER`,
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
    const funGetUsersTask = async (sleep: number) => {
      for (const user of users.value) {
        const tasks = await funGetTaskByUser(user.id)
        const setNums = [...new Set(tasks.map((e) => e?.setNum))]
        for (const num of setNums) {
          const idx = usersTask.value.findIndex((e:any) => e === num)
          if (idx === -1) {
            usersTask.value.push(num)
          }
        }
        // sleep time
        await funTimer(sleep)
      }
    }
    const funGetPics = async () => {
      const allTaskWorkBefore: Array<any> = await funGetTaskByUser('')

      let setNum = 0
      for (const data of allTaskWorkBefore) {
        if (setNum != data.setNum) {
          data.isShow = true
          setNum = data.setNum
        }
      }
      items.value = allTaskWorkBefore
    }
    const funHiddenPics = async () => {
      for (const item of items.value) {
        const idx = usersTask.value.findIndex((e:any) => e === item.setNum)
        if (idx !== -1) {
          item.isShow = false
        }
      }
    }
    const funcNextRequest = async () => {
      const response = await axios.post(
        'https://studio.treed.ai/api/workspace/work/next',
        {
          'taskId': `${params.value.taskId}`,
          'workType': 'OPERATOR'
        },
        {
          headers: {
            'accept-language': 'en-US,en;q=0.9,vi-VN;q=0.8,vi;q=0.7,fr-FR;q=0.6,fr;q=0.5',
            'access-control-allow-origin': '*',
            'access-control-expose-headers': 'Authorization,Content-Disposition,Content-Type',
            'authorization': `${params.value.authorization}`,
            'cookie': 'modalType=update; modalClosed=false;',
            'dnt': '1',
            'origin': 'https://studio.treed.ai',
            'priority': 'u=1, i',
            'referer': `https://studio.treed.ai/workspace/${params.value.taskId}?workType=OPERATOR`,
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
    }
    const funcCatchTask = async () => {
      if (confirm('Bấm OK hoặc bấm ENTER để lấy đầu tiên!')) {
        await funcNextRequest()
      }
    }
    const funMain = async () => {
      await funGetUsers()
      await funGetPics()

      // refresh users new task
      setTimeout(async function () {
        let _sleep = 0
        while(true) {
          await funGetUsersTask(_sleep)
          _sleep = 500
          loading.value = false
        }
      }, 1000)
    }
    const funcCheckOnWishList = (_setNum: number) => {
      const idx = wishList.value.findIndex((e: number) => e === _setNum)
      return idx !== -1
    }
    const funcAddWishList = (_setNum: number) => {
      wishList.value.push(_setNum)
    }
    const funcRemoveWishList = (_setNum: number) => {
      const idx = wishList.value.findIndex((e: number) => e === _setNum)
      wishList.value.splice(idx, 1);
    }
    const funcItemWishList = async () => {
      if (loading.value) return
      const onShowItems = items.value.filter((e: any) => e.isShow === true)
      const idx = wishList.value.findIndex((e: number) => e === onShowItems[0].setNum)
      if (idx !== -1) {
        await funcNextRequest()
        funcRemoveWishList(onShowItems[0].setNum)
      }
    }
    const getChildItem = async (_setNum: number) => {
      childItems.value = []
      childItems.value = items.value.filter((e:any) => e.setNum === _setNum)
    }
    const clearChildItem = async () => {
      childItems.value = []
    }
    return {
      params,
      loading,
      users,
      usersTask,
      items,
      wishList,
      childItems,
      funcItemWishList,
      funMain,
      funTimer,
      funGetUsers,
      funGetTaskByUser,
      funGetUsersTask,
      funcNextRequest,
      funGetPics,
      funHiddenPics,
      funcCatchTask,
      funGetParams,
      funcCheckOnWishList,
      funcAddWishList,
      funcRemoveWishList,
      getChildItem,
      clearChildItem,
    }
  },
  beforeMount() {
    this.funGetParams()
    this.funMain()

    setInterval(()=> {
      this.funHiddenPics()
    }, 1000)

    // check to get wish list
    setInterval(()=> {
      this.funcItemWishList()
    }, 1000)
},
};
</script>
<template>
  <div class="m-4">
    <button
      v-if="loading"
      type="button" class="px-5 text-white bg-blue-700 hover:bg-blue-800 focus:outline-none font-medium rounded-lg px-1 text-center dark:bg-blue-600 dark:hover:bg-blue-700">
      Loading...
    </button>
    <button
      v-else
      type="button" class="px-5 text-white bg-blue-700 hover:bg-blue-800 focus:outline-none font-medium rounded-lg px-1 text-center dark:bg-blue-600 dark:hover:bg-blue-700" @click="funcCatchTask()">
      Lấy Set Đầu
    </button>
    {{ wishList }}

    <div class="mt-4 grid grid-cols-4 gap-2">
      <template v-for="(item, index) in items" :key="index">
        <div v-if="item.isShow">
          <button
            v-if="funcCheckOnWishList(item.setNum)"
            type="button" class="px-5 mb-1 text-white bg-green-700 hover:bg-green-800 focus:outline-none font-medium rounded-lg px-1 text-center dark:bg-green-600 dark:hover:bg-green-700"
            @click="funcRemoveWishList(item.setNum)"
          >
            {{ item.setNum }}
          </button>
          <button
            v-else
            type="button" class="px-5 mb-1 text-white bg-gray-700 hover:bg-gray-800 focus:outline-none font-medium rounded-lg px-1 text-center dark:bg-gray-600 dark:hover:bg-gray-700"
            @click="funcAddWishList(item.setNum)"
          >
            {{ item.setNum }}
          </button>
          <img :src="`https://treed-data-stable.s3.ap-northeast-2.amazonaws.com${item.filePath}`" @click="getChildItem(item.setNum)" >
        </div>
      </template>
    </div>
    <div v-if="childItems.length > 0" tabindex="-1" aria-hidden="true" class="fixed top-[10%] right-0 left-[10%] z-50 justify-center w-[80%]">
      <div class="relative p-4 w-full h-full">
        <div class="relative rounded-lg shadow border-4 bg-gray-300">
          <div class="grid grid-cols-3 gap-2 m-4">
            <template v-for="(childitem, childindex) in childItems" :key="childindex">
              <div>
                <label>{{ childitem.setNum }} - {{ childindex + 1 }} - {{ childitem.itemId }}</label>
                <img :key="childindex" :src="`https://treed-data-stable.s3.ap-northeast-2.amazonaws.com${childitem.filePath}`" >
              </div>
            </template>
          </div>
          <div class="flex items-center p-2 md:p-5 rounded-b dark:border-gray-600">
              <button data-modal-hide="default-modal" type="button" class="py-2.5 px-5 text-sm font-medium text-gray-900 focus:outline-none bg-white rounded-lg border border-gray-200 hover:bg-gray-100 hover:text-blue-700 focus:z-10 focus:ring-4 focus:ring-gray-100 dark:focus:ring-gray-700 dark:bg-gray-800 dark:text-gray-400 dark:border-gray-600 dark:hover:text-white dark:hover:bg-gray-700" @click="clearChildItem()">Bỏ Qua</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
