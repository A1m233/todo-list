<!--
TodoList.vue
-->
<template>
    <div>
        <div style="width: 700px; background-color: black;">
            <el-menu
            class="center-row"
            :default-active="filtered"
            mode="horizontal"
            @select="handleSelect">
                <el-menu-item index="all">
                    <el-icon><Filter /></el-icon>&nbsp;显示全部待办事项
                </el-menu-item>
                <el-menu-item index="incompleted">
                    <el-icon><Filter /></el-icon>&nbsp;显示未完成待办事项
                </el-menu-item>
                <el-menu-item index="completed">
                    <el-icon><Filter /></el-icon>&nbsp;显示已完成待办事项
                </el-menu-item>
            </el-menu>
        </div>
        <div class="center-col">
            <el-form
            class="center-row"
            style="padding-bottom: 25px; padding-top: 25px;"
            @submit.prevent="addNewTodo">
                <el-form-item>
                    <el-input
                    v-model="newTodo"
                    style="width: 240px;"
                    required placeholder="新的待办事项"
                    clearable />
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" native-type="submit">
                        <el-icon><Plus /></el-icon>&nbsp;添加新的待办事项
                    </el-button>
                </el-form-item>
            </el-form>
            <el-button type="primary" @click="exportToCSV">
                <el-icon><Printer /></el-icon>&nbsp;导出全部待办事项，以CSV格式
            </el-button>
        </div>
        <div>
            <VueDraggable v-model="todos" :disabled="filtered !== 'all'">
                <div v-for="(todo, index) in filteredTodos" :key="index">
                    <component
                    :is="Todo"
                    @update-state="newState => updateState(todo.value.id, newState)"
                    @update-content="newContent => updateContent(todo.value.id, newContent)"
                    @delete-todo="deleteTodo(todo.value.id)"
                    :content="todo.value.content"
                    :state="todo.value.state" />
                </div>
            </VueDraggable>
        </div>
    </div>
</template>

<script setup>
    import Todo from './Todo.vue';
    import {computed, inject, ref} from 'vue';
    import Papa from 'papaparse';
    import {saveAs} from 'file-saver';
    import { VueDraggable } from 'vue-draggable-plus';

    const id = inject('id');
    const newTodo = inject('newTodo');
    const updateNewTodo = inject('updateNewTodo');
    let todos = inject('todos');
    const updateTodos = inject('updateTodos');
    const filtered = inject('filtered');
    const updateFiltered = inject('updateFiltered');

    const inputRef = ref(null);

    const filteredTodos = computed(() =>
    {
        if (filtered.value === 'completed')return todos.value.filter(todo => todo.value.state === true);
        else if (filtered.value === 'incompleted')return todos.value.filter(todo => todo.value.state !== true);
        return todos.value;
    });

    // 导出为 CSV 文件，解决中文乱码问题
    function exportToCSV() 
    {
        const data = todos.value.map(todo => (
        {
            '状态': todo.value.state ? '已完成' : '未完成',
            '内容': todo.value.content,
        }));

        const csv = Papa.unparse(data);
        // 添加 UTF-8 BOM 头
        const BOM = '\uFEFF';
        const csvWithBOM = BOM + csv;

        const blob = new Blob([csvWithBOM], { type: 'text/csv;charset=utf-8;' });
        saveAs(blob, 'todos.csv');
    }
    function addNewTodo()
    {
        updateTodos(
        [
            ref(
            {
                id: id.value++,
                type: Todo,
                content: ref(newTodo.value),
                state: ref(false),
            }),
            ...(todos.value),
        ]);
        updateNewTodo('');
    }
    function updateContent(index, newContent) 
    {
        updateTodos(todos.value.map(todo =>
        {
            return todo.value.id === index ? 
            ref(
            {
                ...(todo.value),
                content: ref(newContent),
            }) 
            : todo;
        }));
    }
    function updateState(index, newState)
    {
        updateTodos(todos.value.map(todo =>
        {
            return todo.value.id === index ? 
            ref(
            {
                ...(todo.value),
                state: ref(newState), 
            })
            : todo;
        }));
    }
    function deleteTodo(index)
    {
        updateTodos(todos.value.filter(todo => todo.value.id !== index));
    }
    function handleSelect(index)
    {
        updateFiltered(index);
    }
</script>