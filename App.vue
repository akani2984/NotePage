<template>
  <el-tabs
    v-model="editableTabsValue"
    type="card"
    editable
    @edit="handleTabsEdit"
  >
    <el-tab-pane
      v-for="item in editableTabs"
      :key="item.name"
      :label="item.title"
      :name="item.name"
     
    >
    <editor  :myid="item.name" :title="item.title" @tittlechange="handleTitleChange" ></editor>
    </el-tab-pane>
  </el-tabs>
  <div v-if="!editableTabs[0]">
    <h1>点击右上角的“+”按钮以开始。</h1>
    <p>请不要过度依赖应用内存储。在设备存储空间告急或者七天内不使用本程序（iOS平台）的情况下，您的应用内数据可能会丢失。程序会尽自己所能向浏览器请求持久存储的权限，如果申请成功会弹窗告知。把程序添加到主屏幕，授予通知权限，作为应用安装，频繁使用都有利于程序得到持久存储权限，之后您在本程序的的数据将不会自动清除。Firefox,Chrome,Edge,Safari(IOS)是推荐的浏览器。</p>
    <p>本程序离线可用。</p>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import type { TabPaneName } from 'element-plus'
import  editor  from './components/editor.vue'
import { onMounted } from 'vue';
import { ElMessage } from 'element-plus';
let tabIndex = 0
const editableTabsValue = ref(0)
const editableTabs = ref([])
if (localStorage.getItem('tabcontent')) {
  editableTabs.value = JSON.parse(localStorage.getItem('tabcontent'))
  tabIndex =  JSON.parse(localStorage.getItem('tabIndex'))
  if(editableTabs.value[0]) {
  editableTabsValue.value = editableTabs.value[0].name
  } 
}
const handleTabsEdit = (
  targetName: TabPaneName | undefined,
  action: 'remove' | 'add'
) => {
  if (action === 'add') {
    const newTabName = editableTabs.value.length + 1
    editableTabs.value.push({
      title: '未命名.txt',
      name: newTabName,
      content: '未命名',
    })
    localStorage.setItem('tabcontent', JSON.stringify(editableTabs.value))
    localStorage.setItem('tabIndex', JSON.stringify(tabIndex))
    editableTabsValue.value = newTabName
  } else if (action === 'remove') {
    const tabs = editableTabs.value
    let activeName = editableTabsValue.value
    if (activeName === targetName) {
      tabs.forEach((tab, index) => {
        if (tab.name === targetName) {
          const nextTab = tabs[index + 1] || tabs[index - 1]
          localStorage.removeItem(tab.name)
          if (nextTab) {
            activeName = nextTab.name
          }
        }
      })
    }

    editableTabsValue.value = activeName
    editableTabs.value = tabs.filter((tab) => tab.name !== targetName)
    localStorage.setItem('tabcontent', JSON.stringify(editableTabs.value))
    localStorage.setItem('tabIndex', JSON.stringify(tabIndex))
  }
}
function handleTitleChange(e) {
  editableTabs.value[e[1] -1].title = e[0]
  window.localStorage.setItem('tabcontent', JSON.stringify(editableTabs.value))
}
try {
navigator.storage.persist().then(function (persistent) {
  if (persistent){
      ElMessage('持久存储申请成功！');
  }
})} catch {
  console.log('e')
}
try {
  Notification.requestPermission();
} catch {
  console.log('e')
}
if ('serviceWorker' in navigator) {
  navigator.serviceWorker
    .register('./sw.js') // 指定 Service Worker 文件路径
    .then(registration => console.log('ServiceWorker 注册成功'))
    .catch(err => console.log('注册失败:', err));
}
</script>

<style scoped>
</style>
