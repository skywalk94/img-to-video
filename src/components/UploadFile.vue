<template>
    <ul class="file">
        <li class="file-li" v-for="(item, index) in imgList" :key="index">
            <img class="file-li-main" :src="item.target.result" alt="">
            <img class="file-li-delete" src="../assets/delete.png" alt="" @click="deleteImg(index)" title="删除图片">
        </li>
        <li class="file-li" title="上传(PNG)图片">
            <img class="file-li-pic" src="../assets/pic.png" alt="">
            <p class="file-li-text">上传(PNG)图片</p>
            <input class="file-li-file" type="file" accept="image/png" multiple @change="uploadImg">
        </li>
        <li class="file-li" title="上传背景音乐">
            <img class="file-li-pic" src="../assets/voice.png" alt="">
            <p class="file-li-text">{{ videoObj["name"] ? videoObj.name : '上传背景音乐' }}</p>
            <input class="file-li-file" type="file" accept="audio/*" @change="uploadVideo">
        </li>
    </ul>
</template>

<script setup>
import { ref } from 'vue'
const imgList = ref([])
const videoObj = ref({})

const emit = defineEmits(["change"])


const uploadImg = (e) => {
    let files = e.target.files
    fileReader(files, 0)
}

// 递归获取图片显示
const fileReader = (files, index) => {
    let reader = new FileReader()
    reader.readAsDataURL(files[index])
    reader.onload = (e) => {
        imgList.value.push(Object.assign(e, { raw: files[index] }))
        if (++index < files.length) {
            fileReader(files, index)
        }
        emit("change", { imgs: imgList.value, videoBg: videoObj.value })
    }
}

const uploadVideo = (e) => {
    videoObj.value = e.target.files[0]
    emit("change", { imgs: imgList.value, videoBg: videoObj.value })
}

const deleteImg = (index) => {
    imgList.value.splice(index, 1)
    emit("change", { imgs: imgList.value, videoBg: videoObj.value })
}
</script>

<style lang="less">
.file {
    display: flex;
    align-items: center;
    flex-wrap: wrap;

    &-li {
        position: relative;
        width: 120px;
        height: 120px;
        border: 1px solid #f50057;
        margin: 5px;
        display: flex;
        align-items: center;
        flex-direction: column;
        justify-content: center;
        border-radius: 10px;
        overflow: hidden;

        &-main {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }

        &-pic {
            width: 35px;
        }

        &-delete {
            position: absolute;
            top: 0;
            right: 0;
            width: 20px;
            z-index: 1;
            cursor: pointer;
        }

        &-text {
            margin-top: 15px;
            color: #f50057;
            font-size: 14px;
            width: 110px;
            text-align: center;
            overflow: hidden;
            white-space: nowrap;
            text-overflow: ellipsis;
        }

        &-file {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            opacity: 0;
            font-size: 0;
            cursor: pointer;
        }
    }
}
</style>