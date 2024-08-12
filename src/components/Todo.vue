<!--
Todo.vue
处理单个待办事项的逻辑
-->
<script setup>
    import {ref, watch} from 'vue';
    import {Delete, Edit} from '@element-plus/icons-vue';
    import {ElMessage, ElMessageBox} from 'element-plus';

    const emit = defineEmits(['update-state', 'update-content', 'delete-todo']);
    const props = defineProps(['content' ,'state']);
    const content = ref(props['content']);
    const state = ref(props['state']);
    
    watch(content, newContent =>
    {
        emit('update-content', newContent);
    });
    watch(state, newState =>
    {
        console.log('success');
        emit('update-state', newState);
    });

    function getNewContent()
    {
        ElMessageBox.prompt(
            '请输入新内容',
            '输入框',
            {
                confirmButtonText: '确认',
                cancelButtonText: '取消',
                inputValue: content.value,
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
                content.value = value;
            }
        }).catch(() =>
        {
            ElMessage.info('编辑操作已取消');
        });
    }
</script>

<template>
    <div>
        <el-divider />
        <div>
            <el-checkbox
            v-model="state"
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