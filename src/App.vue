<!--
App.vue
处理tab页切换和页面状态保存与加载逻辑
-->
<template>
    <div class="center-col">
        <el-affix offset="0">
            <el-button type="danger" @click="clearLocalStorage">
                <el-icon><WarnTriangleFilled /></el-icon>&nbsp;清空此待办事项列表。当此待办事项列表出现问题时，可以尝试点击此按钮进行修复
            </el-button>
        </el-affix>
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

<script setup>
    import {onMounted, provide, ref, watch, computed} from 'vue';
    import TodoList from './components/TodoList.vue';
    import Statistics from './components/Statistics.vue';

    const viewList = 
    {
        // 实际上传入的是键值对
        // 这里可以直接放入
        // 是因为属性名和变量名相同
        TodoList,
        Statistics,
    };
    const translate = 
    {
        'TodoList': '待办事项',
        'Statistics': '统计数据',
    };

    const currView = ref('TodoList');
    const todos = ref([]);
    const newTodo = ref('');
    const id = ref(0);
    const filtered = ref('all');

    const completedCount = computed(() =>
    {
        if (!todos.value || !todos.value.length)return 0;
        return todos.value.reduce((accumulator, item) =>
        {
            accumulator += +(item.value.state === true);
            return accumulator;
        }, 0);
    });
    const incompletedCount = computed(() =>
    {
        if (!todos.value || !todos.value.length)return 0;
        return todos.value.reduce((accumulator, item) =>
        {
            accumulator += +(item.value.state !== true);
            return accumulator;
        }, 0);
    });

    provide('todos', todos);
    provide('updateTodos', updateTodos);
    provide('newTodo', newTodo);
    provide('updateNewTodo', updateNewTodo);
    provide('id', id);
    provide('filtered', filtered);
    provide('updateFiltered', updateFiltered);
    provide('completedCount', completedCount);
    provide('incompletedCount', incompletedCount);

    watch(currView, saveToLocalStorage);
    watch(todos, saveToLocalStorage);
    watch(id, saveToLocalStorage);
    watch(newTodo, saveToLocalStorage);
    watch(filtered, saveToLocalStorage);

    function updateTodos(newValue)
    {
        todos.value = newValue;
    }
    function updateFiltered(newValue)
    {
        filtered.value = newValue;
    }
    function updateNewTodo(newValue)
    {
        newTodo.value = newValue;
    }
    function saveToLocalStorage()
    {
        localStorage.setItem('currView', JSON.stringify(currView.value));
        localStorage.setItem('todos', JSON.stringify(todos.value.map(item => item.value)));
        localStorage.setItem('id', JSON.stringify(id.value));
        localStorage.setItem('newTodo', JSON.stringify(newTodo.value));
        localStorage.setItem('filtered', JSON.stringify(filtered.value));
    }
    function loadFromLocalStorage()
    {
        const tmpCurrView = localStorage.getItem('currView');
        currView.value = tmpCurrView ? JSON.parse(tmpCurrView) : 'TodoList';

        const tmpTodos = localStorage.getItem('todos');
        todos.value = tmpTodos ? JSON.parse(tmpTodos) : [];
        todos.value = todos.value.map(item => ref(item));

        const tmpId = localStorage.getItem('id');
        id.value = tmpId ? JSON.parse(tmpId) : 0;

        const tmpNewTodo = localStorage.getItem('newTodo');
        newTodo.value = tmpNewTodo ? JSON.parse(tmpNewTodo) : '';

        const tmpFiltered = localStorage.getItem('filtered');
        filtered.value = tmpFiltered ? JSON.parse(tmpFiltered) : 'all';

        console.log(`App load from localStorage`);
        console.log(`currView: ${currView.value}`);
        console.log(`todos: ${todos.value}`);
        console.log(`id: ${id.value}`);
        console.log(`newTodo: ${newTodo.value}`);
        console.log(`filtered: ${filtered.value}`);
        console.log(`completedCount: ${completedCount.value}`);
        console.log(`incompletedCount: ${incompletedCount.value}`);
    }
    function clearLocalStorage()
    {
        currView.value = 'TodoList';
        todos.value = [];
        newTodo.value = '';
        id.value = 0;
        filtered.value = 'all';
    }
    function handleSelect(key)
    {
        currView.value = key;
    }

    onMounted(() =>
    {
        // 这里如果是第一次启用项目，那么必须先将初始化状态存入localStorage
        // 否则我们会从localStorage中读取null
        // 之前使用!localStorage.length来判断
        // 然而第一次使用时，localStorage却并不是空的

        // 貌似还是存在一个我不清楚的问题
        // 所以直接改成在loadFromLocalStorage()中判断
        loadFromLocalStorage();
    });
</script>
