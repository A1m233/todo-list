<!--
Todo.vue
处理单个待办事项的逻辑
-->
<template>
    <div>
        <el-divider />
        <div>
            <el-checkbox
            v-model="currState"
            :label="content"
            size="large"></el-checkbox>
        </div>
        <div>
            <el-button-group>
                <el-button type="primary" :icon="Edit" @click="getNewContent">编辑内容</el-button>
                <el-popconfirm
                title="是否删除该待办事项？"
                @confirm="emit('delete-todo')"
                :confirmButtonText="'是的'"
                :cancelButtonText="'取消'">
                    <template #reference>
                        <el-button type="primary" :icon="Delete">删除该待办事项</el-button>
                    </template>
                </el-popconfirm>
            </el-button-group>
        </div>
    </div>
</template>

<script setup>
    import {ref, watch, computed} from 'vue';
    import {Delete, Edit} from '@element-plus/icons-vue';
    import {ElMessage, ElMessageBox} from 'element-plus';

    const emit = defineEmits(['update-state', 'update-content', 'delete-todo']);
    const props = defineProps(['content' ,'state']);

    let currState = computed(
    {
        get()
        {
            return props['state'];
        },
        set(newState)
        {
            emit('update-state', newState);
        },
    });
    let inputContent = computed(() => ref(props['content']));
    //watch(props['content'], () => inputContent.value = props['content']);
    function getNewContent()
    {
        ElMessageBox.prompt(
            '请输入新内容',
            '输入框',
            {
                confirmButtonText: '确认',
                cancelButtonText: '取消',
                inputValue: inputContent.value,
            }
        ).then(({value}) =>
        {
            if (value.trim() === '')
            {
                ElMessage.warning('输入内容不应为空');
            }
            else
            {
                ElMessage.success("编辑成功");
                emit('update-content', value);
                //currContent = value;
            }
        }).catch(() =>
        {
            ElMessage.info('编辑操作已取消');
        });
    }
</script>