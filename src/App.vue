<!--
App.vue
-->
<script setup>
    import {onMounted, provide, ref, watch, computed} from 'vue';
    import TodoList from './components/TodoList.vue';
    import Statistics from './components/Statistics.vue';

    const currView = ref('TodoList');
    const viewList = 
    {
        // 实际上传入的是对象
        // 这里可以直接放入是因为组件
        // 是因为属性名和变量名相同
        TodoList,
        Statistics,
    };
    const translate = 
    {
        'TodoList': '待办事项',
        'Statistics': '统计数据',
    };

    const todos = ref([]);
    const newTodo = ref('');
    const id = ref(0);
    const filtered = ref('all');
    const completedCount = computed(() =>
    {
        if (!todos.value || !todos.value.length)return 0;
        return todos.value.reduce((accumulator, item, index, array) =>
        {
            accumulator += +(item.state === true);
            return accumulator;
        }, 0);
    });
    const incompletedCount = computed(() =>
    {
        if (!todos.value || !todos.value.length)return 0;
        return todos.value.reduce((accumulator, item, index, array) =>
        {
            accumulator += +(item.state !== true);
            return accumulator;
        }, 0);
    });
    provide('todos', todos);
    provide('newTodo', newTodo);
    provide('id', id);
    provide('filtered', filtered);
    provide('completedCount', completedCount);
    provide('incompletedCount', incompletedCount);

    watch(currView, saveToLocalStorage);
    watch(todos, saveToLocalStorage);
    watch(id, saveToLocalStorage);
    watch(newTodo, saveToLocalStorage);
    watch(filtered, saveToLocalStorage);

    function saveToLocalStorage()
    {
        localStorage.setItem('currView', JSON.stringify(currView.value));
        localStorage.setItem('todos', JSON.stringify(todos.value));
        localStorage.setItem('id', JSON.stringify(id.value));
        localStorage.setItem('newTodo', JSON.stringify(newTodo.value));
        localStorage.setItem('filtered', JSON.stringify(filtered.value));
    }
    function loadFromLocalStorage()
    {
        const tmp = localStorage.getItem('currView');
        if (tmp)currView.value = JSON.parse(tmp);
        todos.value = JSON.parse(localStorage.getItem('todos'));
        id.value = JSON.parse(localStorage.getItem('id'));
        newTodo.value = JSON.parse(localStorage.getItem('newTodo'));
        filtered.value = JSON.parse(localStorage.getItem('filtered'));
        console.log(`App load from localStorage`);
        console.log(`currView: ${currView.value}`);
        console.log(`todos: ${todos.value}`);
        console.log(`id: ${id.value}`);
        console.log(`newTodo: ${newTodo.value}`);
        console.log(`filtered: ${filtered.value}`);
        console.log(`completedCount: ${completedCount.value}`);
        console.log(`incompletedCount: ${incompletedCount.value}`);
    }
    function handleSelect(key)
    {
        currView.value = key;
    }

    onMounted(() =>
    {
        if (!localStorage.length)
        {
            saveToLocalStorage();
        }
        loadFromLocalStorage();
    });
</script>

<template>
    <div class="center-col">
        <el-affix style="width: 100%;">
            <el-menu
            class="center-row"
            style="min-width: 100%"
            :default-active="currView"
            mode="horizontal"
            @select="handleSelect">
            <el-menu-item v-for="(viewComponent, viewName) in viewList"
            :key="viewName"
            :index="viewName"
            @click="currView = viewName">
                <el-icon v-if="viewName == 'Statistics'"><Histogram /></el-icon>
                <el-icon v-if="viewName == 'TodoList'"><List /></el-icon>
                {{ translate[viewName] }}
            </el-menu-item>
            </el-menu>
        </el-affix>
        <component
        :is="viewList[currView]"/>
        <el-backtop :right="100" :bottom="100" />
    </div>
</template>
