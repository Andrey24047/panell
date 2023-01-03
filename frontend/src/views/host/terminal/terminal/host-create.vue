<template>
    <div>
        <el-dialog v-model="dialogVisiable" :title="$t('terminal.addHost')" width="30%">
            <el-alert
                v-if="isLocal"
                style="margin-bottom: 20px"
                center
                :title="$t('terminal.connLocalErr')"
                :closable="false"
                type="warning"
            />
            <el-form ref="hostRef" label-width="100px" :model="hostInfo" :rules="rules">
                <el-form-item :label="$t('terminal.ip')" prop="addr">
                    <el-input v-if="!isLocal" clearable v-model="hostInfo.addr" />
                    <span v-if="isLocal">{{ hostInfo.addr }}</span>
                </el-form-item>
                <el-form-item :label="$t('terminal.user')" prop="user">
                    <el-input clearable v-model="hostInfo.user" />
                </el-form-item>
                <el-form-item :label="$t('terminal.authMode')" prop="authMode">
                    <el-radio-group v-model="hostInfo.authMode">
                        <el-radio label="password">{{ $t('terminal.passwordMode') }}</el-radio>
                        <el-radio label="key">{{ $t('terminal.keyMode') }}</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item :label="$t('terminal.password')" v-if="hostInfo.authMode === 'password'" prop="password">
                    <el-input clearable show-password type="password" v-model="hostInfo.password" />
                </el-form-item>
                <el-form-item :label="$t('terminal.key')" v-if="hostInfo.authMode === 'key'" prop="privateKey">
                    <el-input clearable type="textarea" v-model="hostInfo.privateKey" />
                </el-form-item>
                <el-form-item :label="$t('terminal.port')" prop="port">
                    <el-input clearable v-model.number="hostInfo.port" />
                </el-form-item>
                <el-form-item :label="$t('commons.table.title')" prop="name">
                    <el-input clearable v-model="hostInfo.name" />
                </el-form-item>
                <el-form-item :label="$t('commons.table.description')" prop="description">
                    <el-input clearable v-model="hostInfo.description" />
                </el-form-item>
            </el-form>
            <template #footer>
                <span class="dialog-footer">
                    <el-button @click="dialogVisiable = false">{{ $t('commons.button.cancel') }}</el-button>
                    <el-button @click="submitAddHost(hostRef, 'testConn')">
                        {{ $t('terminal.testConn') }}
                    </el-button>
                    <el-button type="primary" @click="submitAddHost(hostRef, 'saveAndConn')">
                        {{ $t('terminal.saveAndConn') }}
                    </el-button>
                </span>
            </template>
        </el-dialog>
    </div>
</template>

<script setup lang="ts">
import { ElForm, ElMessage } from 'element-plus';
import { Host } from '@/api/interface/host';
import { Rules } from '@/global/form-rules';
import { addHost, testByInfo } from '@/api/modules/host';
import i18n from '@/lang';
import { reactive, ref } from 'vue';

const dialogVisiable = ref();
type FormInstance = InstanceType<typeof ElForm>;
const hostRef = ref<FormInstance>();

let hostInfo = reactive<Host.HostOperate>({
    id: 0,
    name: '',
    groupBelong: '',
    addr: '',
    port: 22,
    user: '',
    authMode: 'password',
    password: '',
    privateKey: '',
    description: '',
});

const rules = reactive({
    addr: [Rules.requiredInput],
    port: [Rules.requiredInput, Rules.port],
    user: [Rules.requiredInput],
    authMode: [Rules.requiredSelect],
    password: [Rules.requiredInput],
    privateKey: [Rules.requiredInput],
});

const isLocal = ref(false);
interface DialogProps {
    isLocal: boolean;
}
const acceptParams = (props: DialogProps) => {
    isLocal.value = props.isLocal;
    if (props.isLocal) {
        hostInfo.addr = '127.0.0.1';
        hostInfo.user = 'root';
    }
    dialogVisiable.value = true;
};

const emit = defineEmits(['on-conn-terminal', 'load-host-tree']);

const submitAddHost = (formEl: FormInstance | undefined, ops: string) => {
    if (!formEl) return;
    formEl.validate(async (valid) => {
        if (!valid) return;
        hostInfo.groupBelong = 'default';
        switch (ops) {
            case 'testConn':
                await testByInfo(hostInfo).then((res) => {
                    if (res.data) {
                        ElMessage.success(i18n.global.t('terminal.connTestOk'));
                    } else {
                        ElMessage.success(i18n.global.t('terminal.connTestFailed'));
                    }
                });
                break;
            case 'saveAndConn':
                const res = await addHost(hostInfo);
                dialogVisiable.value = false;
                let title = res.data.user + '@' + res.data.addr + ':' + res.data.port;
                if (res.data.name.length !== 0) {
                    title = res.data.name + '-' + title;
                }
                emit('on-conn-terminal', title, res.data.id, res.data.addr, '');
                emit('load-host-tree');
        }
    });
};

defineExpose({
    acceptParams,
});
</script>