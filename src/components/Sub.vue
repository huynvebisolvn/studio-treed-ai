<script lang="ts" setup>
import axios from 'axios';
import { ref, onBeforeMount, onMounted, watch, computed } from "vue";

const params = ref({ authorization: '', projectId: '', taskId: '' })
const loadPicture: any = ref(false)
const waiting: any = ref(true)
const users: any = ref([])
const usersTask: any = ref([])
const myUsersName = ref('')
const usersTaskMap: any = ref(new Map())
const items: any = ref([])
const wishList: any = ref([])
const wishListInput: any = ref('')
const childItems: any = ref([])
const isError = ref(false)

const funTimer = (ms: number) => new Promise((res) => setTimeout(res, ms))
const funGetParams = () => {
  const uri = window.location.search.substring(1);
  const urlParams = new URLSearchParams(uri);
  params.value.authorization = String(urlParams.get("authorization"))
  params.value.projectId = String(urlParams.get("projectId"))
  params.value.taskId = String(urlParams.get("taskId"))

  const _taskId = getCookie('taskId')
  if (!_taskId || params.value.taskId === _taskId) {
    const _myUsersName = getCookie('myUsersName')
    const _wishList = getCookie('wishList')
    // set UsersName
    myUsersName.value = _myUsersName
    // set wishList
    wishListInput.value = _wishList
    funGetWishParams()
  } else {
    setCookie('myUsersName', '')
    setCookie('wishList', '')
  }
  setCookie('taskId', params.value.taskId)
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
  // error
  if (response.data?.code?.includes("TK")) isError.value = true
}
const funGetTaskByUser = async (userId: string, statuses: string): Promise<Array<any>> => {
  const response = await axios.get('https://studio.treed.ai/api/workspace/items', {
    params: {
      'projectId': `${params.value.projectId}`,
      'taskId': `${params.value.taskId}`,
      'workType': 'MANAGER',
      'keyword': '',
      'page': '0',
      'limit': '999999999',
      'statuses': `${statuses}`,
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
  // error
  if (response.data?.code?.includes("TK")) isError.value = true
  return response.data.items
}
const funGetUsersTask = async (userId: string, userName: string) => {
  while (true) {
    try {
      const tasks = await funGetTaskByUser(userId, '')
      if (tasks && tasks[0] && tasks[0].setNum) {
        // setNum
        const setNums = [...new Set(tasks.map((e) => e?.setNum))]
        for (const num of setNums) {
          const idx = usersTask.value.findIndex((e: any) => e === num)
          if (idx === -1) {
            usersTask.value.push(num)
          }
        }
        usersTaskMap.value.set(userName, setNums)
      } else {
        // itemId
        for (const task of tasks) {
          const idx = usersTask.value.findIndex((e: any) => e === task.itemId)
          if (idx === -1) {
            usersTask.value.push(task.itemId)
          }
        }
        const setItemId = [...new Set(tasks.map((e) => e?.itemId))]
        usersTaskMap.value.set(userName, setItemId)
      }
      await funTimer(3000)
    } catch (error) {
      await funTimer(3000)
    }
  }
}
const funGetPics = async () => {
  const allTaskWorkBefore: Array<any> = await funGetTaskByUser('', 'WORK_BEFORE')
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

const firstItem = computed(() => {
  const firstShowIdx = items.value.findIndex((e: any) => e.isShow === true)
  let key = -1
  if (firstShowIdx !== -1) {
    key = items.value[firstShowIdx].setNum
    if (!key) key = items.value[firstShowIdx].itemId
  }
  return key
})

const onNext = async () => {
  // skip when waiting
  if (!waiting.value) {
    console.log(firstItem.value)

    const idx = wishList.value.findIndex((e: number) => e === firstItem.value)
    if (idx !== -1) {
      wishList.value.splice(idx, 1)

      await funTimer(500)
      funcNextRequest()
    }
  }
}

watch(wishList, async () => {
  await onNext()
}, { deep: true })

watch(firstItem, async () => {
  await onNext()
})

watch(usersTask.value, () => {
  for (const item of items.value) {
    if (item.isShow === false) continue

    let key = item.setNum
    if (!key) key = item.itemId

    const idx = usersTask.value.findIndex((e: any) => e === key)
    if (idx !== -1) {
      item.isShow = false
    }
  }
})

onBeforeMount(async () => {
  funGetParams()
})

onMounted(async () => {
  // get first all user and task
  await funGetUsers()
  await funGetPics()

  for (const user of users.value) {
    // refresh users new task
    funGetUsersTask(user.id, user.name)
  }

  // wating for first load done
  await funTimer(10000)
  waiting.value = false

  // 15 minutes will reset task
  setInterval(() => { funGetPics() }, 900000)
})
</script>

<template>
  <div class="m-4">
    <button type="button"
      class="px-5 text-white bg-blue-700 hover:bg-blue-800 focus:outline-none font-medium rounded-lg px-1 text-center dark:bg-blue-600 dark:hover:bg-blue-700"
      @click="loadPicture = !loadPicture">
      {{ waiting ? 'Waiting' : 'Picture' }}
    </button>
    <input class="ml-2 border-2 border-rose-500" v-model="myUsersName" placeholder="Nhập tên user"
      @input="setCookie('myUsersName', myUsersName)" />
    <input class="ml-2 border-2 border-teal-500" v-model="wishListInput" placeholder="Nhập wish list"
      @input="funGetWishParams" />
    <p class="break-all text-rose-500"> {{ usersTaskMap.get(myUsersName) }}</p>
    <p class="break-all text-teal-500">{{ wishList }}</p>
    <label v-if="isError" class="text-3xl text-red-500">Hết hạn rồi, đăng nhập lại!</label>
    <div class="mt-4 grid grid-cols-4 gap-1">
      <template v-for="(item, index) in items" :key="index">
        <div v-if="item.isShow">
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
