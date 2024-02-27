[toc]

---

# Upload-Image

```vue
<script setup lang="ts">
import { ref, h, defineEmits, watch } from "vue"
import type { UploadFileInfo, UploadInst, UploadCustomRequestOptions } from 'naive-ui'
import { fetchCommonUpload } from "@/service/api";
import { $t } from "@/locales";
// import { toBase64 } from "@/utils/videoFrame";

/**
 * @description 图片上传
 * @param options 上传的文件
 * */
interface UploadEmits {
  (e: "success", value: string): void;
  (e: "check-validate"): void;
}
const emit = defineEmits<UploadEmits>();


type FileTypes =
  | "image/apng"
  | "image/bmp"
  | "image/gif"
  | "image/jpeg"
  | "image/pjpeg"
  | "image/png"
  | "image/svg+xml"
  | "image/tiff"
  | "image/webp"
  | "image/x-icon";

interface UploadFileProps {
  defaultUrl: string;
  filelength: number; // 文件数
  fileSize: number; // 文件大小，M
  fileType: FileTypes[];
  disabled: boolean;
  size: string;
  borderRadius: string;
}


// 接受父组件参数
const props = withDefaults(defineProps<UploadFileProps>(), {
  defaultUrl: '',
  disabled: false,
  filelength: 1,
  fileSize: 5,
  fileType: () => ["image/jpeg", "image/png", "image/gif"],
  size: "120px",
  borderRadius: "100%",
});


const fileList = ref<UploadFileInfo[]>([])

const uploadRef = ref<UploadInst | null>()

const fileUrl = ref<string>("");


watch(() => props.defaultUrl, (newValue) => {
  if (fileUrl.value === '') {
    fileUrl.value = newValue;
    console.log(fileUrl.value)
  }
});

async function handleUpload({
  file,
  data,
  headers,
  withCredentials,
  action,
  onFinish,
  onError,
  onProgress
}: UploadCustomRequestOptions) {
  // fileUrl.value = await toBase64(file.file as File)
  const res = await fetchCommonUpload(file.file as File);
  fileUrl.value = res.url
  emit("success", fileUrl.value)
}

function handleBeforeUpload(data: {
  file: UploadFileInfo
  fileList: UploadFileInfo[]
}) {
  const fileInfo = data.file
  const size = fileInfo.file?.size || props.fileSize
  // 检查文件大小
  if (size > props.fileSize * 1024 * 1024) {
    window.$message?.warning(`请上传不超过${props.fileSize}M的文件`);
    return false;
  }
  if (props.fileType.indexOf(fileInfo.type as FileTypes) < 0) {
    window.$message?.warning(`上传图片不符合所需的格式`);
    return false;
  }
  return true;
}

function handleRemove() {
  fileList.value = []
}

function handleView() {
  window.$dialog?.info({
    title: $t('common.preview'),
    content: () => {
      return h(
        "img",
        { src: fileUrl.value, class: 'w-100% object-cover' }
      )
    },
    maskClosable: true,
    showIcon: false
  });
}
</script>

<template>
  <n-upload ref="uploadRef" abstract v-model:file-list="fileList" :custom-request="handleUpload"
    :accept="props.fileType.join(',')" @before-upload="handleBeforeUpload" :disabled="props.disabled">
    <div :style="['width:' + props.size, 'height:' + props.size]"
      class="border-dotted border-1 bordre-#c0ccda border-rd-2  position-relative cursor-pointer overflow-hidden">
      <n-upload-trigger #="{ handleClick }" abstract class="size-full position-relative">
        <div class="size-full position-relative" @click="handleClick" v-if="fileUrl === ''">
          <icon-local-plus class="text-24px position-absolute absolute top-50% left-50% translate--50%" />
        </div>
        <div :style="['border-radius:' + props.borderRadius]"
          class="size-full mr-10px position-absolute overflow-hidden top-50% left-50% translate--50%" v-else>
          <img :src="fileUrl" class="size-full" />
          <div
            class="position-absolute top-0 size-full  z-index-10 opacity-0 hover:opacity-100 bg-#f9f9f999 transition-all-300 transition-ease-linear">
            <div class="position-absolute top-50% left-50% translate--50% w-80% flex flex-row justify-around">
              <icon-local-view class="text-24px" @click="handleView" />
              <icon-local-upload class="text-24px" @click="() => { handleRemove(); handleClick() }" />
              <!-- <icon-local-delete class="text-24px" @click="handleRemove" /> -->
            </div>
          </div>
        </div>
      </n-upload-trigger>
    </div>
  </n-upload>
</template>

<style scoped></style>

```

