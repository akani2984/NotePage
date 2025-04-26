<template>
<input ref="input" type="file" style="display:none ;" @change="handleFileOpen" ></input>
<el-dropdown placement="bottom-start">
    <el-button> 菜单 </el-button>
    <template #dropdown>
      <el-dropdown-menu>
        <el-dropdown-item @click="openFile">打开</el-dropdown-item>
        <el-dropdown-item @click="saveDialogVisible = true" >保存/更改标题</el-dropdown-item>
      </el-dropdown-menu>
    </template>
  </el-dropdown>
  <el-dropdown placement="bottom-start">
    <el-button> 查看 </el-button>
    <template #dropdown>
      <el-dropdown-menu>
        <el-dropdown-item @click="handleReplaceUtils">查找/替换</el-dropdown-item>
      </el-dropdown-menu>
    </template>
</el-dropdown>
<br>
<div v-show="replaceUtil">
    <el-input v-model="searchValue" ref="searchInput" style="width: 100vh" placeholder="查找值" />
    <el-input v-model="replaceValue" style="width: 100vh" placeholder="替换值" />
    <br>
    <div style="display: flex;">
        <el-button @click="handleSearchPrevious" >查找上一个</el-button>
        <el-button @click="handleSearchNext" >查找下一个</el-button>
        <el-button @click="handleReplace">替换</el-button>
        <el-button @click="handleReplaceAll">全部替换</el-button>
    </div>
</div>
<el-input
    v-model="textcontent"
    style="width: 100%;"
    :rows="textRows"
    type="textarea"
    ref="textarea"
  />
  <el-dialog
    v-model="saveDialogVisible"
    title="保存/更改标题"
    style="width: 100%"
    
  >
    <span>请输入名称</span>
    <el-input v-model="filename"></el-input>
    <template #footer>
      <div class="dialog-footer">
        <el-button @click="saveDialogVisible = false">取消</el-button>
        <el-button @click="handleRename">变更标题</el-button>
        <el-button type="primary" @click="handleSave">
          确定
        </el-button>
      </div>
    </template>
  </el-dialog>
  <a :download="filename" style="display: none;" ref="a" href=''></a>
 
</template>
<script setup> 
import { ref } from 'vue';
import { onMounted } from 'vue';
import { ElMessage } from 'element-plus';
const textcontent =ref('');
const textarea = ref(null);
const textRows = ref(0);
const input = ref(null);
const tabname = ref('');
const a = ref(null);
const emit = defineEmits(['tittlechange']);
const saveDialogVisible = ref(false);
const filename = ref('');
const prop = defineProps({
  myid: Number,
  title: String,
});
const replaceUtil = ref(false);
const searchValue = ref('');
const replaceValue = ref('');
let searchCount = -1;
const searchInput = ref(null);

function handleresize() {
    let vheight = Math.max(
    document.documentElement.clientHeight || 0,
    window.innerHeight || 0,
    window.visualViewport?.height || 0) * 0.7
    textRows.value = Math.floor(Number(vheight) / Number(window.getComputedStyle(textarea.value.textarea).lineHeight.slice(0, -2)));
}
function openFile() {
    input.value.click();
}
function handleFileOpen(e) {
    let file = e.target.files[0];
    emit('tittlechange',  [file.name, prop.myid]);
    filename.value = file.name
    const reader = new FileReader();
    reader.readAsText(file);
    reader.addEventListener('load', () => {textcontent.value = reader.result; localStorage.setItem(prop.myid, reader.result)} );

}
function handleSave() {
    emit('tittlechange',  [filename.value, prop.myid] );
    let blob = new Blob([textcontent.value]);
    a.value.href = window.URL.createObjectURL(blob)
    a.value.click()
    saveDialogVisible.value = false
}
function handletextchange() {
    localStorage.setItem(prop.myid, textcontent.value)
}
function handleRename() {
    emit('tittlechange',  [filename.value, prop.myid] );
    saveDialogVisible.value = false
}
function handleReplaceUtils() {
    replaceUtil.value = !replaceUtil.value 
}
function handleSearchNext() {
    let test = handleSearch(searchCount++)
    if (test == 0) {
        handleSearch(searchCount--)
    } 
    if (test == false) {
        return
    }
}
function handleSearch() {
    const content = textarea.value.textarea.value;
    const ele = textarea.value.textarea;
    let startPos = -1;
    for (let i = -1; i <=  searchCount; i++ ) {
        startPos = content.indexOf(searchValue.value, startPos + 1); 
        if(startPos == -1 && i === -1) {
            ElMessage('未找到值！');
            break
        }
        if (i === searchCount) {
            ele.setSelectionRange(startPos, startPos + searchValue.value.length);
            ele.focus();
            ele.scrollTop = ele.scrollHeight;
            return true
        }
    }
}
function handleSearchPrevious() {
    console.log(searchCount)
    if (searchCount <= -1) {
        handleSearch(searchCount)
    } else {
    handleSearch(searchCount--)
    }
}
function handleReplace() {
    let temptextarea = textarea.value.textarea;
    let newText = replaceValue.value
    const start = temptextarea.selectionStart;
    const end = temptextarea.selectionEnd;
    const original = temptextarea.value; 
    textcontent.value = original.substring(0, start) + 
                   newText + 
                   original.substring(end);
    temptextarea.selectionStart = temptextarea.selectionEnd = start + newText.length;
    temptextarea.focus();
    handletextchange();
}
function handleReplaceAll() {
    textcontent.value = textcontent.value.replaceAll(searchValue.value, replaceValue.value);
    handletextchange();
}
function handleSearchInput() {
    searchCount = -1;
}
onMounted(() => {
    handleresize();
    window.addEventListener('resize', handleresize);
    textarea.value.textarea.addEventListener('input', handletextchange);
    searchInput.value.input.addEventListener('input', handleSearchInput);
})
filename.value = prop.title
if (localStorage.getItem(prop.myid)) {
textcontent.value = localStorage.getItem(prop.myid)
}

</script>
<style scoped>
</style>