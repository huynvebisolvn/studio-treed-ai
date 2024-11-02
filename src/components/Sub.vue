<script lang="ts" setup>
import axios from 'axios';
import { ref, onBeforeMount, onMounted } from "vue";

const params = ref({ authorization: '', projectId: '', taskId: '' })
const loadPicture: any = ref(false)
const nextIdx = ref()
const items: any = ref([])
const wishList: any = ref([])
const wishListInput: any = ref('')
const childItems: any = ref([])
const isError = ref(false)

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
  // error
  if (response.data?.code?.includes("TK")) isError.value = true
  return response.data.operators
}

const funGetTaskByUser = async (userIds?: Array<string>): Promise<Array<any>> => {
  const idsParam = userIds ? userIds.join(',') : ''
  const response = await axios.get('https://studio.treed.ai/api/workspace/items', {
    params: {
      'projectId': `${params.value.projectId}`,
      'taskId': `${params.value.taskId}`,
      'workType': 'MANAGER',
      'keyword': '',
      'page': '0',
      'limit': '999999999',
      'statuses': '',
      'operatorIds': `${idsParam}`
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
  // error
  if (response.data?.code?.includes("TK")) isError.value = true
  return response.data.items
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
  console.log(response)
}

const funGetWishParams = () => {
  const wishs = wishListInput.value?.split(',')
  let rs = []
  if (wishs) {
    for (const wish of wishs) {
      if (Number(wish)) rs.push(Number(wish))
    }
  }
  wishList.value = rs
  setCookie('wishList', rs.join(','))
}

const funGetParams = () => {
  const uri = window.location.search.substring(1);
  const urlParams = new URLSearchParams(uri);
  params.value.authorization = String(urlParams.get("authorization"))
  params.value.projectId = String(urlParams.get("projectId"))
  params.value.taskId = String(urlParams.get("taskId"))

  const _taskId = getCookie('taskId')
  if (!_taskId || params.value.taskId === _taskId) {
    const _wishList = getCookie('wishList')
    // set wishList
    wishListInput.value = _wishList
    funGetWishParams()
  } else {
    setCookie('wishList', '')
  }
  setCookie('taskId', params.value.taskId)
}

const funTimer = (ms: number) => new Promise((res) => setTimeout(res, ms))

const funGetUsersTask = async (userIds: Array<string>) => {
  while (true) {
    try {
      const tasks = await funGetTaskByUser(userIds)
      let idx = undefined
      if (tasks && tasks[0] && tasks[0].setNum) {
        // setNum
        const maxKey = Math.max(...tasks.map(o => o.setNum))
        console.log("đã chọn đến: ", maxKey)
        idx = items.value.findIndex(e => e.setNum === maxKey)
      } else {
        // itemId
        const maxKey = Math.max(...tasks.map(o => o.itemId))
        console.log("đã chọn đến: ", maxKey)
        idx = items.value.findIndex(e => e.itemId === maxKey)
      }
      if (idx) {
        // check next item on wish
        nextIdx.value = idx + 1
        const isOnWish = funcCheckOnWishList(items.value[idx + 1])
        if (isOnWish) {
          wishList.value.splice(idx, 1)
          await funcNextRequest()
        }
      }
      await funTimer(2000)
    } catch (error) { }
  }
}
const funGetPics = async () => {
  const allTaskWorkBefore: Array<any> = await funGetTaskByUser(undefined)
  if (allTaskWorkBefore && allTaskWorkBefore[0] && allTaskWorkBefore[0].setNum) {
    // has setNum
    let setNum = 0
    for (const data of allTaskWorkBefore) {
      if (setNum != data.setNum) {
        data.isShow = true
        setNum = data.setNum
      }
    }
  } else {
    // don't has setNum
    for (const data of allTaskWorkBefore) {
      data.isShow = true
    }
  }
  // allTaskWorkBefore.filter((e) => !e.operatedDate)
  items.value = allTaskWorkBefore
}

const funcCheckOnWishList = (item: any) => {
  let key = item.setNum
  if (!key) key = item.itemId

  const idx = wishList.value.findIndex((e: number) => e === key)
  return idx !== -1
}

const funcAddWishList = (item: any) => {
  let key = item.setNum
  if (!key) key = item.itemId

  wishList.value.push(key)
}

const funcRemoveWishList = (item: any) => {
  let key = item.setNum
  if (!key) key = item.itemId

  const idx = wishList.value.findIndex((e: number) => e === key)
  wishList.value.splice(idx, 1);
}

const getChildItem = (item: any) => {
  childItems.value = []

  let key = item.setNum
  if (key) {
    childItems.value = items.value.filter((e: any) => e.setNum === key)
  }
}

const clearChildItem = () => {
  childItems.value = []
}

const setCookie = (cname: string, cvalue: string) => {
  document.cookie = cname + '=' + encodeURIComponent(cvalue) + '; path=/';
}

const getCookie = (cname: string) => {
  const regex = new RegExp('(?:^|; )' + encodeURIComponent(cname) + '=([^;]*)');
  const match = document.cookie.match(regex);
  return match ? decodeURIComponent(match[1]) : '';
}

onBeforeMount(() => {
  funGetParams()
})

onMounted(async () => {
  // get first all user and task
  const users = await funGetUsers()
  await funGetPics()

  const ids = [...new Set(users.map((e) => e.id))]
  funGetUsersTask(ids)
})
</script>

<template>
  <div class="m-4">
    <button type="button"
      class="px-5 text-white bg-blue-700 hover:bg-blue-800 focus:outline-none font-medium rounded-lg px-1 text-center dark:bg-blue-600 dark:hover:bg-blue-700"
      @click="loadPicture = !loadPicture">
      Picture
    </button>
    <input class="ml-2 border-2 border-teal-500" v-model="wishListInput" placeholder="Nhập wish list"
      @input="funGetWishParams" />
    <p class="break-all text-teal-500">{{ wishList }}</p>
    <label v-if="isError" class="text-3xl text-red-500">Hết hạn rồi, đăng nhập lại!</label>
    <div class="mt-4 grid grid-cols-4 gap-1">
      <template v-for="(item, index) in items" :key="index">
        <div v-if="item.isShow && index >= nextIdx">
          <button v-if="funcCheckOnWishList(item)" type="button"
            class="px-3 mb-1 text-white bg-green-700 hover:bg-green-800 focus:outline-none font-medium rounded-lg px-1 text-center dark:bg-green-600 dark:hover:bg-green-700"
            @click="funcRemoveWishList(item)">
            {{ item.setNum ? item.setNum : item.itemId }}
          </button>
          <button v-else type="button"
            class="px-3 mb-1 text-white bg-gray-700 hover:bg-gray-800 focus:outline-none font-medium rounded-lg px-1 text-center dark:bg-gray-600 dark:hover:bg-gray-700"
            @click="funcAddWishList(item)">
            {{ item.setNum ? item.setNum : item.itemId }}
          </button>
          <img v-if="loadPicture" :src="`https://treed-data-stable.s3.ap-northeast-2.amazonaws.com${item.filePath}`"
            @click="getChildItem(item)">
        </div>
      </template>
    </div>
    <div v-if="childItems.length > 0" tabindex="-1" aria-hidden="true"
      class="fixed top-0 right-0 left-[10%] z-50 justify-center w-[80%]">
      <div class="relative p-4 w-full h-full">
        <div class="relative rounded-lg shadow border-4 bg-gray-300">
          <div class="grid grid-cols-4 gap-2 m-4 overflow-auto" style="height: 750px;">
            <template v-for="(childitem, childindex) in childItems" :key="childindex">
              <div>
                <label>{{ childitem.setNum }} - {{ childindex + 1 }} - {{ childitem.itemId }}</label>
                <img :key="childindex"
                  :src="`https://treed-data-stable.s3.ap-northeast-2.amazonaws.com${childitem.filePath}`">
              </div>
            </template>
          </div>
          <div class="flex items-center p-2 md:p-5 rounded-b dark:border-gray-600">
            <button data-modal-hide="default-modal" type="button"
              class="py-2.5 px-5 text-sm font-medium text-gray-900 focus:outline-none bg-white rounded-lg border border-gray-200 hover:bg-gray-100 hover:text-blue-700 focus:z-10 focus:ring-4 focus:ring-gray-100 dark:focus:ring-gray-700 dark:bg-gray-800 dark:text-gray-400 dark:border-gray-600 dark:hover:text-white dark:hover:bg-gray-700"
              @click="clearChildItem()">Bỏ Qua</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
