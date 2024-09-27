<script setup lang="ts">

import { ref, computed } from 'vue'
let title = ref('PSD文件预览');
import { InboxOutlined } from '@ant-design/icons-vue';
import type { UploadChangeParam, UploadProps } from 'ant-design-vue';

const fileList = ref<UploadProps['fileList']>([]);

const beforeUpload: UploadProps['beforeUpload'] = file => {
  fileList.value.push(file)
  console.log(fileList.value);
  fileList.value.forEach((file) => {
    // 读取文件内容为 ArrayBuffer
    const reader = new FileReader();
    reader.readAsArrayBuffer(file);
    // 监听 FileReader 的 load 事件
    reader.onload = (event) => {
      // 获取 ArrayBuffer 数据
      const arrayBuffer = reader.result;
      const psd = readPsd(arrayBuffer);
      document.body.appendChild(psd.children[0].canvas);
      console.log('psd===', psd);
      // 现在你可以使用这个 ArrayBuffer
      // 例如，转换为十六进制表示或其他操作
    };
    // 错误处理
    reader.onerror = (error) => {
      console.error('读取文件时出错:', error);
    };
  })
  return false;
};

// 自定义上传逻辑（不上传到服务器）
const customRequest = ({ file, onSuccess }) => {
  const reader = new FileReader();
  reader.onload = (e) => {
    const result = e.target?.result;
    if (result) {
      // 这里可以对文件进行后续处理
      console.log('File content:', result);

      // 调用 onSuccess 表示上传成功
      onSuccess('success');
    }
  };
  reader.onerror = (error) => {
    console.error('Error reading file:', error);
  };
  reader.readAsDataURL(file);
};
const handleChange = (info: UploadChangeParam) => {
  // console.log('handleChange', info);
};
function handleDrop(e: DragEvent) {
  console.log(e);
}

import { readPsd } from 'ag-psd';
</script>

<template>
  <div class="PsdViewer">
    <h1>
      {{ title }}
    </h1>
    <a-upload-dragger v-model:fileList="fileList" name="file" accept=".psd" :multiple="true"
      :beforeUpload="beforeUpload" @change="handleChange" @drop="handleDrop">
      <p class="ant-upload-drag-icon">
        <inbox-outlined></inbox-outlined>
      </p>
      <p class="ant-upload-text">单击或拖动文件到此区域进行上传</p>
      <p class="ant-upload-hint">
        只支持xxx.psd文件
      </p>
    </a-upload-dragger>
  </div>
</template>

<style scoped>
.PsdViewer {
  width: 100%;
}
</style>
