<template>
    <div v-loading="loading">
        <el-row :gutter="20">
            <el-col :span="12">
                <el-input v-model="req.name" @blur="searchByName"></el-input>
            </el-col>
            <el-col :span="11">
                <el-select v-model="req.tags" multiple style="width: 100%" @change="changeTag">
                    <el-option v-for="item in tags" :key="item.key" :label="item.name" :value="item.key"></el-option>
                </el-select>
            </el-col>
            <el-col :span="1">
                <el-button @click="sync">{{ $t('app.sync') }}</el-button>
            </el-col>
            <el-col v-for="(app, index) in apps" :key="index" :xs="8" :sm="8" :lg="4">
                <div @click="getAppDetail(app.id)">
                    <el-card :body-style="{ padding: '0px' }" class="a-card">
                        <el-row :gutter="24">
                            <el-col :span="8">
                                <div class="icon">
                                    <el-image class="image" :src="'data:image/png;base64,' + app.icon"></el-image>
                                </div>
                            </el-col>
                            <el-col :span="16">
                                <div class="a-detail">
                                    <div class="d-name">
                                        <font size="3" style="font-weight: 700">{{ app.name }}</font>
                                    </div>
                                    <div class="d-description">
                                        <font size="1">
                                            <span>
                                                {{ app.shortDesc }}
                                            </span>
                                        </font>
                                    </div>
                                    <div class="d-tag">
                                        <el-tag v-for="(tag, ind) in app.tags" :key="ind" round :colr="getColor(ind)">
                                            {{ tag.name }}
                                        </el-tag>
                                    </div>
                                </div>
                            </el-col>
                        </el-row>
                    </el-card>
                </div>
            </el-col>
        </el-row>
    </div>
</template>

<script lang="ts" setup>
import { App } from '@/api/interface/app';
import { onMounted, reactive, ref } from 'vue';
import router from '@/routers';
import { SearchApp, SyncApp } from '@/api/modules/app';
import { ElMessage } from 'element-plus';
import i18n from '@/lang';

let req = reactive({
    name: '',
    tags: [],
    page: 1,
    pageSize: 50,
});

let apps = ref<App.App[]>([]);
let tags = ref<App.Tag[]>([]);
const colorArr = ['#6495ED', '#54FF9F', '#BEBEBE', '#FFF68F', '#FFFF00', '#8B0000'];
let loading = ref(false);

const getColor = (index: number) => {
    return colorArr[index];
};

const search = async (req: App.AppReq) => {
    await SearchApp(req).then((res) => {
        apps.value = res.data.items;
        tags.value = res.data.tags;
    });
};

const getAppDetail = (id: number) => {
    let params: { [key: string]: any } = {
        id: id,
    };
    router.push({ name: 'AppDetail', params });
};

const sync = () => {
    loading.value = true;
    SyncApp()
        .then(() => {
            ElMessage.success(i18n.global.t('app.syncSuccess'));
            search(req);
        })
        .finally(() => {
            loading.value = false;
        });
};

const changeTag = () => {
    search(req);
};

const searchByName = () => {
    search(req);
};

onMounted(() => {
    search(req);
});
</script>

<style lang="scss">
.header {
    padding-bottom: 10px;
}

.a-card {
    height: 100px;
    margin-top: 10px;
    cursor: pointer;
    padding: 5px;

    .icon {
        width: 100%;
        height: 80%;
        padding: 10%;
        margin-top: 5px;
        .image {
            width: auto;
            height: auto;
        }
    }

    .a-detail {
        margin-top: 10px;
        height: 100%;
        width: 100%;

        .d-name {
            height: 20%;
        }

        .d-description {
            overflow: hidden;
            display: -webkit-box;
            -webkit-line-clamp: 2;
            -webkit-box-orient: vertical;
        }
    }
}

.a-card:hover {
    transform: scale(1.1);
}
</style>