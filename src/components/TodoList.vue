<!--
TodoList.vue
-->
<script setup>
    import Todo from './Todo.vue';
    import {computed, inject} from 'vue';
    import Papa from 'papaparse';
    import {saveAs} from 'file-saver';

    const id = inject('id');
    const newTodo = inject('newTodo');
    const todos = inject('todos');
    const filtered = inject('filtered');

    const filteredTodos = computed(() =>
    {
        if (filtered.value === 'completed')return todos.value.filter(todo => todo.state === true);
        else if (filtered.value === 'incompleted')return todos.value.filter(todo => todo.state !== true);
        return todos.value;
    });

    // 导出为 CSV 文件，解决中文乱码问题
    function exportToCSV() 
    {
        const data = todos.value.map(todo => (
        {
            '状态': todo.state ? '已完成' : '未完成',
            '内容': todo.content,
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
        console.log(todos);
        todos.value.push(
        {
            id: id.value++,
            type: Todo,
            content: newTodo.value,
            state: false, 
        });
        newTodo.value = '';
    }
    function updateContent(index, newContent) 
    {
        todos.value = todos.value.map(todo =>
        {
            return todo.id === index ? 
            {
                ...todo,
                content: newContent,
            } 
            : todo;
        });
    }
    function updateState(index, newState)
    {
        todos.value = todos.value.map(todo =>
        {
            return todo.id === index ? 
            {
                ...todo,
                state: newState, 
            }
            : todo;
        });
    }
    function deleteTodo(index)
    {
        todos.value = todos.value.filter(todo => todo.id !== index);
        console.log(todos);
    }
    function handleSelect(index)
    {
        filtered.value = index;
    }
</script>

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
        <div
        class="center-row"
        style="padding-bottom: 25px; padding-top: 25px;">
            <el-input v-model="newTodo" style="width: 240px;" required placeholder="新的待办事项" />
            <el-button type="primary" @click="addNewTodo">
                <el-icon><Plus /></el-icon>&nbsp;添加新的待办事项
            </el-button>
        </div>
        <div class="center-row">
            <el-button type="primary" @click="exportToCSV">
                <el-icon><Printer /></el-icon>&nbsp;导出全部待办事项，以CSV格式
            </el-button>
        </div>
        <div>
            <ul>
                <li v-for="todo in filteredTodos" :key="todo.id">
                    <KeepAlive>
                        <component
                        :is="Todo"
                        @update-state="newState => updateState(todo.id, newState)"
                        @update-content="newContent => updateContent(todo.id, newContent)"
                        @delete-todo="deleteTodo(todo.id)"
                        :content="todo.content"
                        :state="todo.state">
                            {{todo.content}}
                        </component>
                    </KeepAlive>
                </li>
            </ul>
        </div>
    </div>
</template>