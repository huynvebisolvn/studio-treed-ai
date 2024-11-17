<script lang="ts" setup>
import axios from 'axios'
import { useInterval } from '@vueuse/core'
import { vOnClickOutside } from '@vueuse/components'
import { ref, onBeforeMount, onMounted, computed, watch } from "vue"

const { counter, pause, resume } = useInterval(1000, { controls: true })
const REFRESH_TIME = 600

const params = ref({ authorization: '', projectId: '', taskId: '', user: '' })
const loadPicture: any = ref(false)
const items: any = ref([])
const wishList: any = ref([])
const wishListInput: any = ref('')
const childItems: any = ref([])
const myTaskList: any = ref([])
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
    }
  })
  // error
  if (response.data?.code?.includes("TK")) isError.value = true
  return response.data.operators
}

const funGetTaskByUser = async (userIds?: Array<string>, status?: string): Promise<Array<any>> => {
  const idsParam = userIds ? userIds.join(',') : ''
  const response = await axios.get('https://studio.treed.ai/api/workspace/items', {
    params: {
      'projectId': `${params.value.projectId}`,
      'taskId': `${params.value.taskId}`,
      'workType': 'MANAGER',
      'keyword': '',
      'page': '0',
      'limit': '999999999',
      'statuses': `${status}`,
      'operatorIds': `${idsParam}`
    },
    headers: {
      'accept': 'application/json, text/plain, */*',
      'accept-language': 'en-US,en;q=0.9,vi-VN;q=0.8,vi;q=0.7,fr-FR;q=0.6,fr;q=0.5',
      'access-control-allow-origin': '*',
      'access-control-expose-headers': 'Authorization,Content-Disposition,Content-Type',
      'authorization': `${params.value.authorization}`,
      'content-type': 'application/json',
    }
  })
  // error
  if (response.data?.code?.includes("TK")) isError.value = true
  const items = response.data.items
  if (items && items[0] && items[0].setNum) {
    return items.sort(function (a, b) {
      return (a.setNum ? a.setNum : Infinity) - (b.setNum ? b.setNum : Infinity)
    })
  }
  return items
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
        'content-type': 'application/json',
      }
    }
  )
  console.log(response)
}

const funGetWishParams = () => {
  const wishs = wishListInput.value?.replaceAll('[', '').replaceAll(']', '').split(',')
  let rs = []
  if (wishs) {
    for (const wish of wishs) {
      if (Number(wish)) rs.push(Number(wish))
    }
  }
  wishList.value = rs
}

const funGetParams = () => {
  const uri = window.location.search.substring(1)
  const urlParams = new URLSearchParams(uri)
  params.value.authorization = String(urlParams.get("authorization"))
  params.value.projectId = String(urlParams.get("projectId"))
  params.value.taskId = String(urlParams.get("taskId"))
  params.value.user = String(urlParams.get("user"))

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

const funGetUsersTask = async (myIds?: number) => {
  while (true) {
    try {
      let originalTasks = await funGetPics('WORK_BEFORE')

      const users = await funGetUsers()
      const userIds = [...new Set(users.map((e: any) => e.id))]

      const tasks = await funGetTaskByUser(userIds)

      let assignedTask = []
      if (tasks && tasks[0] && tasks[0].setNum) {
        // setNum
        assignedTask = [...new Set(tasks.map((e: any) => e.setNum))]
        for (const orgTask of originalTasks) {
          const idx = assignedTask.findIndex((e: any) => e === orgTask.setNum)
          if (idx !== -1) {
            orgTask.isShow = false
          }
        }
      } else {
        // itemId
        assignedTask = [...new Set(tasks.map((e: any) => e.itemId))]
        for (const orgTask of originalTasks) {
          const idx = assignedTask.findIndex((e: any) => e === orgTask.itemId)
          if (idx !== -1) {
            orgTask.isShow = false
          }
        }
      }

      // update new item
      items.value = originalTasks

      // important: get task
      const firstItem = originalTasks.find((e: any) => e.isShow)
      console.log(firstItem.setNum ? firstItem.setNum : firstItem.itemId)

      if (firstItem) {
        // check next item on wish
        let key = firstItem.setNum
        if (!key) key = firstItem.itemId

        const idx = wishList.value.findIndex((e: number) => e === key)
        if (idx !== -1) {
          wishList.value.splice(idx, 1)
          await funcNextRequest()

          // no await get my task
          if (myIds) {
            funGetMyTask(myIds)
          } 
        }
      }
      await funTimer(3000)
    } catch (error) {
      console.log(error)
    }
  }
}

const funGetMyTask = async (userId: number) => {
  const tasks = await funGetTaskByUser([String(userId)])
  myTaskList.value = [...new Set(tasks.map((e: any) => e.setNum ? e.setNum : e.itemId))]
}

const funGetPics = async (status: string) => {
  const tasks: Array<any> = await funGetTaskByUser(undefined, status)
  if (tasks && tasks[0] && tasks[0].setNum) {
    // has setNum
    let setNum = 0
    for (const task of tasks) {
      if (setNum != task.setNum) {
        task.isShow = true
        setNum = task.setNum
      }
    }
  } else {
    // don't has setNum
    for (const task of tasks) {
      task.isShow = true
    }
  }
  return tasks
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
  wishList.value.splice(idx, 1)
}

const togglePicture = () => {
  loadPicture.value = !loadPicture.value
  if (!loadPicture.value) {
    resume()
  } else {
    pause()
  }
}

const getChildItem = (item: any) => {
  childItems.value = []

  let key = item.setNum
  if (key) {
    childItems.value = items.value.filter((e: any) => e.setNum === key)
  } else {
    childItems.value.push(item)
  }
}

const clearChildItem = () => {
  childItems.value = []
}

const setCookie = (cname: string, cvalue: string) => {
  document.cookie = cname + '=' + encodeURIComponent(cvalue) + '; path=/'
}

const getCookie = (cname: string) => {
  const regex = new RegExp('(?:^|; )' + encodeURIComponent(cname) + '=([^;]*)')
  const match = document.cookie.match(regex)
  return match ? decodeURIComponent(match[1]) : ''
}

watch(counter, () => {
  // refresh page every time
  if (counter.value >= REFRESH_TIME && !loadPicture.value) {
    window.location.reload()
  }
})

watch(wishList, () => {
  setCookie('wishList', wishList.value.join(','))
}, { deep: true })

onBeforeMount(() => {
  funGetParams()
})

onMounted(async () => {
  // show item in screen
  items.value = await funGetPics('WORK_BEFORE')

  // get my task first time
  const users = await funGetUsers()
  const myUserId = users.find((e: any) => e.name === params.value.user)?.id
  if (myUserId) {
    funGetMyTask(myUserId)
  }

  // loop for fetch task
  funGetUsersTask(myUserId)
})
</script>

<template>
  <div class="m-4">
    <div class="flex flex-row">
      <div class="basis-1/2">
        <button type="button"
          class="px-5 text-white bg-blue-700 hover:bg-blue-800 focus:outline-none font-medium rounded-lg px-1 text-center dark:bg-blue-600 dark:hover:bg-blue-700"
          @click="togglePicture">
          Picture
        </button>
        <input class="ml-2 border-2 border-teal-500 w-1/2" v-model="wishListInput" placeholder="Nhập wish list" @input="funGetWishParams" />
      </div>      
      <div class="basis-1/2 text-right text-rose-500">{{ REFRESH_TIME - counter }}</div>
    </div>
    <p class="break-all text-rose-500">{{ myTaskList }}</p>
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

          <div :style="`height: ${ loadPicture ? 320 : 5 }px; width: 400px;`" class="bg-gray-300">
            <img
              v-if="loadPicture"
              :key="item.filePath"
              loading="eager"
              style="height: 320px; width: 400px;"
              :src="`https://treed-data-stable.s3.ap-northeast-2.amazonaws.com${item.filePath}`"
              @click="getChildItem(item)"
            />
          </div>
        </div>
      </template>
    </div>

    <div v-if="childItems.length > 1" tabindex="-1" aria-hidden="true" class="fixed top-0 z-50 justify-items-center w-full h-full">
      <div class="relative overflow-auto rounded-lg shadow bg-gray-200 h-full" v-on-click-outside="clearChildItem">
        <div>Tổng cộng: {{ childItems.length }}</div>
        <div v-for="(childitem, childindex) in childItems" :key="childindex" class="mb-2">
          <img loading="eager" :src="`https://treed-data-stable.s3.ap-northeast-2.amazonaws.com${childitem.filePath}`" />
        </div>
      </div>
    </div>

    <div v-if="childItems.length === 1" tabindex="-1" aria-hidden="true" class="fixed top-0 left-0 z-50 justify-items-center w-full h-full">
      <div>
        <img
          class="relative rounded-lg shadow border-4 bg-gray-200 h-full"
          :key="childItems[0]?.filePath"
          loading="eager"
          :src="`https://treed-data-stable.s3.ap-northeast-2.amazonaws.com${childItems[0]?.filePath}`"
          v-on-click-outside="clearChildItem"
        />
      </div>
    </div>
  </div>
</template>
