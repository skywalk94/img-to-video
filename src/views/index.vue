<template>
    <div class="container">
        <UploadFile @change="changeFile" />
        <div class="form">
            <button @click="imgToVideo" class="form-btn" :disabled="imgs.length > 0 && isFinish ? false : true">
                {{ imgs.length > 0 ? "合成视频" : "☝️请上传(PNG)图片" }}
            </button>
            <input class="form-input" v-model="frameNumber" type="number" placeholder="设置帧数(默认1)">
        </div>
        <p class="text" v-if="loggerText">{{ isFinish ? "合成视频完成" : loggerText }}</p>
        <video class="main" :src="videoUrl" controls v-if="videoUrl"></video>
    </div>
</template>
<script setup>
import { ref } from 'vue'
import UploadFile from "@/components/UploadFile.vue"
import FFmpeg from "@ffmpeg/ffmpeg"
const { createFFmpeg, fetchFile } = FFmpeg
const ffmpeg = createFFmpeg({ log: false })

const imgs = ref([])
const videoBg = ref({})
const changeFile = (e) => {
    imgs.value = e.imgs
    videoBg.value = e.videoBg
}

const videoUrl = ref("") //视频链接
const loggerText = ref("") //合成日志
const isFinish = ref(true) // 是否合成完成
const frameNumber = ref("") // 帧数

const imgToVideo = async () => {
    if (!ffmpeg.isLoaded()) {
        await ffmpeg.load()
    }
    for (let i in imgs.value) {
        ffmpeg.FS('writeFile', `${i}.png`, await fetchFile(imgs.value[i].raw))
    }

    // 读取进度
    ffmpeg.setProgress(({ ratio }) => {
        isFinish.value = ratio == 1 ? true : false
    })
    // 读取消息日志
    ffmpeg.setLogger((res) => {
        loggerText.value = res.message
    })

    const count = frameNumber.value.toString() || '1' //视频帧数
    const time = (imgs.value.length / count).toString() //视频时长,不刻意设置也能符合预期，只是重新合成视频会是之前的时长

    if (videoBg.value['name']) {
        ffmpeg.FS('writeFile', videoBg.value.name, await fetchFile(videoBg.value))
        await ffmpeg.run('-r', count, '-f', 'image2', '-i', '%d.png', '-i', videoBg.value.name, '-t', time, 'video.mp4')
    } else {
        await ffmpeg.run('-r', count, '-f', 'image2', '-i', '%d.png', '-t', time, 'video.mp4')
    }
    const data = ffmpeg.FS('readFile', 'video.mp4')
    videoUrl.value = URL.createObjectURL(new Blob([data.buffer], { type: 'video/mp4' }))
}
</script>
<style lang="less">
.container {
    padding: 30px;
    display: flex;
    align-items: center;
    flex-direction: column;
    justify-content: center;
}

.form {
    display: flex;
    align-items: center;
    margin-top: 30px;

    &-btn {
        color: #fff;
        background-color: #f50057;
        font-size: 14px;
        border-radius: 4px;
        height: 36px;
        padding: 5px 15px;
    }

    &-input {
        color: #fff;
        font-size: 14px;
        border-radius: 4px;
        height: 36px;
        padding: 5px 15px;
        outline: unset;
        border: 1px solid #f50057;
        width: 135px;
        margin-left: 10px;
        background: transparent;

        &::-webkit-input-placeholder {
            color: #999;
        }

        &::-webkit-outer-spin-button,
        &::-webkit-inner-spin-button {
            -webkit-appearance: none;
        }

        &[type="number"] {
            -moz-appearance: textfield;
        }
    }
}

.main {
    max-width: 350px;
    margin-top: 35px;
}

.text {
    margin-top: 10px;
}
</style>
