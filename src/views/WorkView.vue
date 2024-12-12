<template>
    <el-row justify="center">
        <el-col :span="18">
            <el-button type="primary" @click="dialogFormVisible = true">添加任务</el-button>
        </el-col>
    </el-row>
    <el-row justify="center">
        <el-col :span="18">
            <el-table v-loading="tableLoading" :data="tableData" style="width: 100%" height="460px">
                <el-table-column prop="id" label="任务id" sortable width="180" />
                <el-table-column prop="name" label="任务描述" sortable width="180" />
                <el-table-column fixed="right" label="操作" min-width="120">
                    <template #default="{ row }">
                        <el-button link type="danger" size="small" @click="handleDetail(row)">
                            删除
                        </el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-col>
    </el-row>
    <el-dialog v-model="dialogFormVisible" title="Shipping address" width="500" align-center>
        <div>
            <el-form ref="form" :model="workForm" label-width="80px">
                <el-form-item label="任务选择">
                    <el-select v-model="workForm.id" placeholder="请选择任务">
                        <el-option v-for="item in tasks" :key="item" :label="item.label" :value="item.value" />
                    </el-select>
                </el-form-item>
                <el-form-item label="任务描述">
                    <el-input v-model="workForm.name" />
                </el-form-item>
                <el-form-item label="任务类型">
                    <el-select v-model="workForm.type" placeholder="请选择任务类型">
                        <el-option v-for="item in taskTypes" :key="item" :label="item.label" :value="item.value" />
                    </el-select>
                </el-form-item>
                <el-form-item v-if="workForm.type == 'interval'" label="间隔时间">
                    <el-row justify="space-around">
                        <el-col :span="7">
                            <el-input v-model="workForm.hour" aria-placeholder="输入小时"><template
                                    #append>时</template></el-input>
                        </el-col>
                        <el-col :span="7">
                            <el-input v-model="workForm.minute" aria-placeholder="输入分钟"><template
                                    #append>分</template></el-input>
                        </el-col>
                        <el-col :span="7">
                            <el-input v-model="workForm.second" aria-placeholder="输入秒"><template
                                    #append>秒</template></el-input>
                        </el-col>
                    </el-row>
                </el-form-item>
                <el-form-item v-if="workForm.type == 'date'" label="执行日期">
                    <el-date-picker v-model="workForm.date" type="datetime" placeholder="Select date and time" />
                </el-form-item>
                <el-form-item v-if="workForm.type == 'interval'" label="执行次数">
                    <el-select v-model="workForm.loop" placeholder="请选择执行次数">
                        <el-option label="一次" value="0" />
                        <el-option label="无限次" value="1" />
                    </el-select>
                </el-form-item>
            </el-form>
        </div>
        <template #footer>
            <div class="dialog-footer">
                <el-button @click="dialogFormVisible = false">Cancel</el-button>
                <el-button type="primary" @click="addWork">
                    Confirm
                </el-button>
            </div>
        </template>
    </el-dialog>
</template>

<script lang="ts" setup>
import { reactive, ref, computed, onMounted, onBeforeMount } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { listTasks, deleteTask, addTask } from '../axios/api'
import { id } from 'element-plus/es/locale';
const route = useRoute()
const router = useRouter()

const tableLoading = ref(false)
const tableData = ref([])
const dialogFormVisible = ref(false)
const taskTypes = [
    {
        label: '循环执行',
        value: 'interval'
    },
    {
        label: '指定时间执行',
        value: 'date'
    }
];  // 更具描述性的命名
const tasks = [
    {
        value:"get_new_index_daily",
        label: '获取指数数据'
    },
    {
        value:"get_new_stock_data",
        label: '获取股票数据'
    },
    {
        value:"refresh_new_stock_data",
        label: '刷新股票数据'
    }
];
const workForm = ref({
    id: '',
    name: '',
    type: 'interval',  // 默认选中 'interval'
    hour: '',
    minute: '',
    second: '',
    date: 1609459200000,
    loop: '0'
});
// 格式化日期为 "yyyy-MM-dd HH:mm:ss"
const formatDate = (date) => {
    if (!date) return ''; // 如果没有日期，返回空字符串

    const d = new Date(date);
    const year = d.getFullYear();
    const month = String(d.getMonth() + 1).padStart(2, '0'); // 月份从0开始，需要+1
    const day = String(d.getDate()).padStart(2, '0');
    const hours = String(d.getHours()).padStart(2, '0');
    const minutes = String(d.getMinutes()).padStart(2, '0');
    const seconds = String(d.getSeconds()).padStart(2, '0');

    return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
};

// 构建目标格式数据
const taskData = computed(() => {
    return {
        id: workForm.value.id,
        name: workForm.value.name,
        type: workForm.value.type,
        hour: Number(workForm.value.hour),  // 转为数字
        minute: Number(workForm.value.minute),  // 转为数字
        second: Number(workForm.value.second),  // 转为数字
        date: formatDate(workForm.value.date),  // 格式化日期
        loop: Number(workForm.value.loop)
    };
});



const handleDetail = async (row) => {
    try {
        const result = await deleteTask(row.id)
        if (result.message === 1) {
            tableData.value = result.tasks;
            tableLoading.value = true;
            const result1 = await listTasks()
            tableData.value = result1.tasks;
            tableLoading.value = false;
        } else {
            console.log(result.message)
        }
    } catch (error) {
        console.log(error)
    } finally {

    }
}

const addWork = async () => {
    try {
        const result = await addTask(taskData.value)
        console.log(result);
        tableLoading.value = true;
        const result1 = await listTasks()
        tableData.value = result1.tasks;
        //清空workForm
        workForm.value = {
            id: '',
            name: '',
            type: 'interval',  // 默认选中 'interval'
            hour: '',
            minute: '',
            second: '',
            date: 1609459200000,
            loop: '0'
        };
        tableLoading.value = false;
    } catch (error) {
        console.log(error)
    } finally {
        dialogFormVisible.value = false
    }
}

onMounted(async () => {
    tableLoading.value = true
    try {
        const result = await listTasks();
        if (result) {
            tableData.value = result.tasks;
        }
        console.log(tableData.value)
    } catch (error) {
        console.log(error)
    } finally {
        tableLoading.value = false
    }
})
</script>