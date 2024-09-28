<script setup lang="ts">

import { ref } from 'vue'
let title = ref('PSD文件预览');
import { InboxOutlined } from '@ant-design/icons-vue';
import type { UploadProps } from 'ant-design-vue';

import { readPsd } from 'ag-psd';

const fileList = ref<any[]>([]);
type imagesInfo = { uid: string, imgUrl: string, layerUrl: Array<string> }
const imagesList = ref<Array<imagesInfo>>([]);
const beforeUpload: UploadProps['beforeUpload'] = file => {
  if (fileList.value == undefined) {
    fileList.value = []
  }
  fileList.value.push(file);
  // 读取文件内容为 ArrayBuffer
  const reader = new FileReader();
  // new Blob()
  reader.readAsArrayBuffer(file);
  // 监听 FileReader 的 load 事件
  reader.onload = () => {
    // 获取 ArrayBuffer 数据
    const arrayBuffer = reader.result;
    if (arrayBuffer) {
      const psd = readPsd(arrayBuffer as ArrayBuffer);
      imagesList.value.push({
        uid: file.uid, imgUrl: psd.canvas?.toDataURL() || '', layerUrl: (psd.children?.map(item => {
          return item.canvas?.toDataURL() || ''
        }) || []).reverse()
      });
    }
  };
  // 错误处理
  reader.onerror = (error) => {
    console.error('读取文件时出错:', error);
  };
  return false;
};
const removeFile: UploadProps['onRemove'] = (file) => {
  console.log('remove', file);
  const uid = file.uid;
  // 找到要删除对象的索引
  const index = imagesList.value.findIndex(obj => obj.uid === uid);
  if (index !== -1) {
    // 使用 splice 方法删除对象
    imagesList.value.splice(index, 1);
  }
}

function handleDrop(e: DragEvent) {
  console.log(e);
}

// 图片预览
let visible = ref(false)
let selectedItem = ref<imagesInfo>({
  uid: '',
  imgUrl: '',
  layerUrl: []
})
const imagePreview = (item: imagesInfo) => {
  selectedItem.value = item;
  visible.value = true
}
</script>

<template>
  <div class="PsdViewer">
    <h1>
      {{ title }}
    </h1>
    <a-upload-dragger v-model:fileList="fileList" name="file" accept=".psd" :multiple="true"
      :beforeUpload="beforeUpload" @drop="handleDrop" @remove="removeFile">
      <p class="ant-upload-drag-icon">
        <inbox-outlined></inbox-outlined>
      </p>
      <p class="ant-upload-text">单击或拖动文件到此区域进行上传</p>
      <p class="ant-upload-hint">
        只支持xxx.psd文件
      </p>
    </a-upload-dragger>
    <a-image v-for="item in imagesList" :key="item.uid" :preview="{ visible: false }" :width="200" :src="item.imgUrl"
      @click="imagePreview(item)" />
    <div style="display: none">
      <a-image-preview-group :preview="{ visible, onVisibleChange: (vis: boolean) => (visible = vis) }">
        <a-image key="origin" :src="selectedItem.imgUrl"></a-image>
        <a-image v-for="(layerUrl, index) in selectedItem.layerUrl" :key="index" :src="layerUrl" />
      </a-image-preview-group>
    </div>
  </div>
</template>

<style scoped>
.PsdViewer {
  width: 100%;
}
</style>
