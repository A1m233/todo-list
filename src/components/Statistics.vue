<!--
Statistics.vue
展示统计数据
-->
<script setup>
    import {inject} from 'vue';

    const completed = inject('completedCount');
    const incompleted = inject('incompletedCount');
    /*
    我们可以直接使用props中的名字，在模版中调用它们
    也就是说completedCount等效于props['completedCount']
    但是，在调用时务必加上.value
    因为当 ref 作为响应式数组或原生集合类型 (如 Map) 中的元素被访问时，它不会被解包
    */
</script>

<template>
    <div class="center-row">
        <el-card>
            <el-statistic :value="completed">
                <template #title>
                    <div>
                        已完成的待办事项总数
                    </div>
                </template>
            </el-statistic>
        </el-card>
        <el-card>
            <el-statistic :value="incompleted">
                <template #title>
                    <div>
                        未完成的待办事项总数
                    </div>
                </template>
            </el-statistic>
        </el-card>
        <el-card style="min-width: 160px;">
            <el-statistic :value="incompleted + completed">
                <template #title>
                    <div>
                        待办事项总数
                    </div>
                </template>
            </el-statistic>
        </el-card>
    </div>
</template>
