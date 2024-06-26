<template>
  <div class="demo">
    <a-layout style="height: 100%">
      <a-layout-sider
        breakpoint="lg"
        collapsed-width="0"
        style="padding: 20px 10px; background: #fff"
      >
        <div>
          <a-space direction="vertical" style="width: 100%">
            <!-- 官方/非官方 -->
            <a-radio-group
              v-model:value="officialKey"
              size="default"
              button-style="solid"
              @change="onOfficialChange"
              style="width: 100%"
            >
              <a-radio-button
                :value="item.value"
                style="width: 50%"
                v-for="(item, index) in officialTabs"
                :key="item.value"
                >{{ item.text }}</a-radio-button
              >
            </a-radio-group>
            <!-- 2D/3D -->
            <a-radio-group
              v-model:value="gameModeKey"
              size="default"
              style="width: 100%"
              @change="gameModalChange"
            >
              <a-radio-button
                :value="item.value"
                style="width: 50%"
                v-for="(item, index) in gameModeTabs"
                :key="item.value"
                >{{ item.text }}</a-radio-button
              >
            </a-radio-group>
            <a-input-search
              v-model:value="searchVal"
              :placeholder="t('demo.search.placeholder')"
              @search="onSearch"
            />
          </a-space>
        </div>

        <a-menu
          v-model:selectedKeys="selectedKeys"
          v-model:openKeys="openKeys"
          theme="light"
          mode="inline"
          @click="handleClick"
          style="border: 0"
          v-if="menuShowList.length"
        >
          <template v-for="item in menuShowList" :key="item.id">
            <RecursiveMenu :item="item" />
          </template>
        </a-menu>
        <div class="empty" v-else>
          <a-empty />
        </div>
      </a-layout-sider>
      <a-layout>
        <a-layout-content :style="{ margin: '24px 10px 0' }">
          <a-space direction="vertical" style="width: 100%" v-if="dataDetail">
            <a-card :title="dataDetail.title">
              <template #extra>
                <a-space style="width: 100%">
                  <a-tooltip placement="bottom" color="white">
                    <template #title>
                      <a-qrcode :value="dataDetail.qrLink" />
                    </template>
                    <a-button size="small">
                      <template #icon> <qrcode-outlined /> </template>
                      {{ t('demo.handleBtns.qrcode') }}</a-button
                    >
                  </a-tooltip>

                  <a-button size="small" @click="openCodeModal">
                    <template #icon> <eye-outlined /> </template>
                    {{ t('demo.handleBtns.viewCode') }}</a-button
                  >
                  <a-button size="small" type="primary" @click="downloadCode">
                    <template #icon> <DownloadOutlined /> </template>

                    {{ t('demo.handleBtns.downloadCode') }}</a-button
                  >
                </a-space>
              </template>

              <iframe :src="dataDetail.demoLink" style="width: 100%; height: 74vh"></iframe>
            </a-card>
            <!-- <div :style="{ padding: '24px', background: '#fff', minHeight: '360px' }">content</div> -->
            <a-card>
              <div>{{ dataDetail.detail }}</div>
            </a-card>
          </a-space>
          <div class="empty" v-else>
            <a-empty description="请点击左侧选择" />
          </div>
        </a-layout-content>
      </a-layout>
    </a-layout>
    <CodeSeeModal ref="codeSeeModalRef" />
  </div>
</template>
<script lang="ts" setup>
import { Modal, type MenuProps, type RadioChangeEvent } from 'ant-design-vue'
import { i18n } from '@/locales/setupI18n'
import { GameMode, OfficialType } from './enums'
import RecursiveMenu from '@/components/RecursiveMenu.vue'
import type { Menu } from '@/components/interface'
import CodeSeeModal from './components/CodeSeeModal.vue'
import { getDataById, getMunusByType } from './controller/menuHandle'
import { downloadFile, deepFilter } from '@/utils/tools'

interface DataDetail {
  title: string 
  demoLink: string 
  codeLink: string 
  qrLink: string 
  detail: string 
}

/**
 * 国际化
 */
const t = i18n.global.t as (key: string) => string
/**
 *  菜单列表
 */
const menuList = ref<Menu[]>([])
const menuShowList = ref<Menu[]>([])
/**
 * 代码查看弹窗
 */
const codeSeeModalRef = ref<InstanceType<typeof CodeSeeModal>>()
/**
 * 详情数据
 */
const dataDetail = ref<DataDetail | null>(null)

/**
 *  官方类型tabs
 */
const officialTabs = ref([
  {
    text: t('demo.officialTabs.official'),
    value: OfficialType.OFFICIAL
  },
  {
    text: t('demo.officialTabs.unOfficial'),
    value: OfficialType.UNOFFICIAL
  }
])
/**
 *  游戏模式tabs
 */
const gameModeTabs = ref([
  {
    text: t('demo.gameModeTabs.twoD'),
    value: GameMode.TWO_D
  },
  {
    text: t('demo.gameModeTabs.threeD'),
    value: GameMode.THREE_D
  }
])
/**
 * 搜索框值
 */
const searchVal = ref<string>('')
/**
 * 官方类型key
 */
const officialKey = ref<OfficialType>(OfficialType.OFFICIAL)
/**
 * 游戏模式key
 */
const gameModeKey = ref<GameMode>(GameMode.TWO_D)

/**
 * 重置
 */
const reset = () => {
  dataDetail.value = null
  searchVal.value = ''
}
/**
 * 官方/非官方切换
 * @param e
 */
function onOfficialChange(e: RadioChangeEvent) {
  // const value: OfficialType = e.target.value
  gameModeKey.value = GameMode.TWO_D
  reset()
  getList()
}
/**
 *  游戏模式切换
 * @param e
 */
function gameModalChange(e: RadioChangeEvent) {
  reset()
  getList()
}
/**
 * 选中的key
 */
const selectedKeys = ref<string[]>([])
/**
 * 展开的key
 */
const openKeys = ref<string[]>([])
/**
 * 搜索
 * @param searchValue
 */
const onSearch = (searchValue?: string) => {
  menuShowList.value = deepFilter(menuList.value, (item) => {
    return (
      !!item.label?.toLocaleLowerCase().includes(searchVal.value.toLocaleLowerCase()) ||
      !searchVal.value
    )
  })
}
/**
 * 点击事件
 */
const handleClick: MenuProps['onClick'] = (e) => {
  const detail = getDataById(e.key as string)
  if (!detail) return
  dataDetail.value = detail
}
/**
 * 获取列表
 */
function getList() {
  menuList.value = getMunusByType({
    officialType: officialKey.value,
    gameMode: gameModeKey.value
  }) as Menu[]
  menuShowList.value = menuList.value
}

/**
 * 打开代码弹窗
 */
function openCodeModal() {
  console.log(codeSeeModalRef.value)

  if (dataDetail.value?.codeLink && codeSeeModalRef.value) {
    codeSeeModalRef?.value.openModal(dataDetail.value?.codeLink)
  } else {
    Modal.warning({
      title: '暂无代码查看'
      // content: 'some messages...some messages...'
    })
  }
}
/**
 * 下载代码
 
 */
function downloadCode() {
  if (dataDetail.value?.codeLink) {
    const fileName = new Date().getTime() + '.js'
    downloadFile(dataDetail.value?.codeLink, fileName)
  }
}
onMounted(() => {
  getList()
})
</script>
<style lang="less" scoped>
::v-deep(.ant-menu-item, .ant-menu) {
  border-radius: 0;
}
.demo {
  height: 100%;
}
#components-layout-demo-responsive .logo {
  height: 32px;
  background: rgba(255, 255, 255, 0.2);
  margin: 16px;
}

.site-layout-sub-header-background {
  background: #fff;
}

.site-layout-background {
  background: #fff;
}

[data-theme='dark'] .site-layout-sub-header-background {
  background: #141414;
}
.empty {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
