<template>
  <aside class="nav">
    <ul class="nav-list">
      <li
        class="nav-item flex-center"
        v-for="(nav, idx) in navInfoList"
        :key="idx + nav.uuid"
        :class="{ active: nav.isActive }"
        @click="onNavClick(nav)"
        v-show="nav.isShow"
      >
        <div class="nav-content">
          <el-icon :size="navIconSize">
            <component :is="nav.icon"></component>
          </el-icon>
          <span class="nav-name">{{ nav.name }}</span>
        </div>
      </li>
    </ul>
  </aside>
</template>

<script setup lang="ts">
import { computed, onMounted, ref, triggerRef, watch } from 'vue'
import { useRouter } from 'vue-router'
import { useStore } from '@/store'
import { ElementPlusSizeEnum, ThemeModeEnum } from '@/common/model'
import { navInfoList } from './nav-content.data'
import { isDarkModeOfSystem } from '@/utils'

const router = useRouter()
const store = useStore()

const userConfigInfo = computed(() => store.getters.getUserConfigInfo).value
const userSettings = computed(() => store.getters.getUserSettings).value

const navIconSize = computed(() => {
  switch (userSettings.elementPlusSize) {
    case ElementPlusSizeEnum.small:
      return 22
    case ElementPlusSizeEnum.large:
      return 30
    default:
      return 26
  }
})

const isDarkMode = ref<boolean>(false)

const onNavClick = (e: any) => {
  const { path } = e

  if (path === '/management') {
    if (userConfigInfo.selectedRepo === '') {
      ElMessage.warning('Choose a repository')
      router.push('/config')
      return
    }

    if (userConfigInfo.selectedDir === '') {
      ElMessage.warning('Dictionary can not be empty')
      router.push('/config')
      return
    }
  }
  router.push(path)
}

const changeNavActive = (currentPath: string) => {
  navInfoList.value.forEach((v) => {
    const temp = v
    temp.isActive = v.path === currentPath || currentPath.includes(v.path)
    return temp
  })

  triggerRef(navInfoList)
}

const persistUserSettings = () => {
  store.dispatch('USER_SETTINGS_PERSIST')
}

const themeModeChange = (e: boolean) => {
  userSettings.theme.mode = e ? ThemeModeEnum.dark : ThemeModeEnum.light
  persistUserSettings()
}

watch(
  () => router.currentRoute.value,
  (_n) => {
    changeNavActive(_n.path)
  }
)

watch(
  () => userConfigInfo.logined,
  (_n) => {
    navInfoList.value.forEach((v: any) => {
      // eslint-disable-next-line default-case
      switch (v.path) {
        case '/management':
        case '/settings':
          // eslint-disable-next-line no-param-reassign
          v.isShow = _n
      }
    })
  },
  {
    deep: true,
    immediate: true
  }
)

watch(
  () => userSettings.theme.mode,
  (_n) => {
    if (_n === ThemeModeEnum.follow) {
      isDarkMode.value = isDarkModeOfSystem()
    } else isDarkMode.value = _n === ThemeModeEnum.dark
  },
  {
    deep: true,
    immediate: true
  }
)

onMounted(() => {
  router.isReady().then(() => {
    changeNavActive(router.currentRoute.value.path)
  })
})
</script>

<style scoped lang="stylus">
@import "nav-content.styl"
</style>
