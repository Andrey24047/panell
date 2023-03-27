<template>
    <el-drawer v-model="drawerVisiable" :destroy-on-close="true" :close-on-click-modal="false" size="50%">
        <template #header>
            <DrawerHeader :header="$t('firewall.ipRule')" :back="handleClose" />
        </template>
        <div v-loading="loading">
            <el-form ref="formRef" label-position="top" :model="dialogData.rowData" :rules="rules">
                <el-row type="flex" justify="center">
                    <el-col :span="22">
                        <el-form-item :label="$t('firewall.address')" prop="address">
                            <el-input
                                :autosize="{ minRows: 3, maxRows: 6 }"
                                type="textarea"
                                clearable
                                v-model="dialogData.rowData!.address"
                            />
                        </el-form-item>
                        <el-form-item :label="$t('firewall.strategy')" prop="strategy">
                            <el-radio-group v-model="dialogData.rowData!.strategy">
                                <el-radio label="accept">{{ $t('firewall.accept') }}</el-radio>
                                <el-radio label="drop">{{ $t('firewall.drop') }}</el-radio>
                            </el-radio-group>
                        </el-form-item>
                    </el-col>
                </el-row>
            </el-form>
        </div>
        <template #footer>
            <span class="dialog-footer">
                <el-button @click="drawerVisiable = false">{{ $t('commons.button.cancel') }}</el-button>
                <el-button type="primary" @click="onSubmit(formRef)">
                    {{ $t('commons.button.confirm') }}
                </el-button>
            </span>
        </template>
    </el-drawer>
</template>

<script lang="ts" setup>
import { reactive, ref } from 'vue';
import { Rules } from '@/global/form-rules';
import i18n from '@/lang';
import { ElForm } from 'element-plus';
import DrawerHeader from '@/components/drawer-header/index.vue';
import { MsgSuccess } from '@/utils/message';
import { Host } from '@/api/interface/host';
import { operateIPRule } from '@/api/modules/host';

const loading = ref();

interface DialogProps {
    title: string;
    rowData?: Host.RulePort;
    getTableList?: () => Promise<any>;
}
const title = ref<string>('');
const drawerVisiable = ref(false);
const dialogData = ref<DialogProps>({
    title: '',
});
const acceptParams = (params: DialogProps): void => {
    dialogData.value = params;
    title.value = i18n.global.t('commons.button.' + dialogData.value.title);
    drawerVisiable.value = true;
};
const emit = defineEmits<{ (e: 'search'): void }>();

const handleClose = () => {
    drawerVisiable.value = false;
};

const rules = reactive({
    address: [Rules.requiredInput],
});

type FormInstance = InstanceType<typeof ElForm>;
const formRef = ref<FormInstance>();

const onSubmit = async (formEl: FormInstance | undefined) => {
    if (!formEl) return;
    formEl.validate(async (valid) => {
        if (!valid) return;
        loading.value = true;
        dialogData.value.rowData.operation = 'add';
        if (!dialogData.value.rowData) return;
        await operateIPRule(dialogData.value.rowData);
        loading.value = false;
        MsgSuccess(i18n.global.t('commons.msg.operationSuccess'));
        emit('search');
        drawerVisiable.value = false;
    });
};

defineExpose({
    acceptParams,
});
</script>