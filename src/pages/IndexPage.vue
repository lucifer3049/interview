<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" :rules="[val => val && val.length > 0 || 'Please type name']" />
        <q-input v-model="tempData.age" label="年齡" :rules="[
          val => val !== null && val !== '' || 'Please type your age',
          val => val > 0 && val < 100 || 'Please type a real age'
        ]" />
        <q-btn color="primary" class="q-mt-md" @click="addNewPersonnel">新增</q-btn>
      </div>
      <!--blockData  -->
      <q-table flat bordered ref="tableRef" :rows="personnel" :columns="(tableConfig as QTableProps['columns'])"
        row-key="id" hide-pagination separator="cell" :rows-per-page-options="[0]">
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td v-for="col in props.cols" :key="col.name" :props="props" style="min-width: 120px">
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn @click="handleClickOption(btn, props.row)" v-for="(btn, index) in tableButtons" :key="index"
                size="sm" color="grey-6" round dense :icon="btn.icon" class="q-ml-md" padding="5px 5px">
                <q-tooltip transition-show="scale" transition-hide="scale" anchor="top middle" self="bottom middle"
                  :offset="[10, 10]">
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div class="full-width row flex-center items-center text-primary q-gutter-sm" style="font-size: 18px">
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref, onMounted } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
]);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);
const personnel = ref([]);

const tempData = ref({
  name: '',
  age: '',
});
function handleClickOption(btn, data) {
  if (btn.status === 'delete') {
    deletePersonnel(data.id);
  } else if (btn.status === 'edit') {
    editPersonnel(data);

  }
}
// 查詢人員
const getPersonnel = async () => {
  try {
    const response = await axios.get(`https://dahua.metcfire.com.tw/api/CRUDTest/a`)
    personnel.value = response.data;
    console.log(personnel.value);

  } catch (error) {
    console.log(error);

  }
}
// 新增人員
const addNewPersonnel = async () => {
  try {
    const response = await axios.post(`https://dahua.metcfire.com.tw/api/CRUDTest`, tempData.value);
    console.log(response.data);
    getPersonnel();
  } catch (error) {
    alert(error)
  }
}
// 編輯人員
const editPersonnel = async () => {
  try {
    await axios.patch(`https://dahua.metcfire.com.tw/api/CRUDTest/${tempData.value}`, {
      headers: {
        'Content-Type': 'application/json'
      }
    });
    // 編輯成功後重新獲取所有人員數據
    getPersonnel();
  } catch (error) {
    console.error("編輯操作失敗:", error);
    // 考慮添加更友好的錯誤處理機制
  }
};
// 刪除人員
const deletePersonnel = async (id) => {
  try {
    await axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${id}`);
    // 刪除成功後重新獲取所有人員數據
    alert("是否確定刪除該筆資料?")
    getPersonnel();
  } catch (error) {
    console.error("刪除操作失敗:", error);
  }
};


onMounted(() => {
  getPersonnel();
});
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
