<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" :rules="[requiredRule]" />
        <q-input
          v-model="tempData.age"
          label="年齡"
          :rules="[requiredRule, positiveIntegerRule]"
        />
        <q-btn color="primary" class="q-mt-md" @click="addNewRow">新增</q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
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
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <template v-if="props.row.editIng">
                <q-input v-model="props.row[col.field]" dense />
              </template>
              <template v-else>
                <div>{{ col.value }}</div>
              </template>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
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
interface eachRowDataType {
  age: number;
  id: string;
  name: string;
  creatorId: string;
  editIng: boolean;
}
onMounted(async () => {
  await searchData();
});
// 查找資料
async function searchData() {
  try {
    const response = await axios.get(
      'https://demo.mercuryfire.com.tw:49110/crudTest/a'
    );
    if (response.status === 200) {
      blockData.value = response.data.result;
      blockData.value.forEach((data) => {
        data.editIng = false;
      });
    }
  } catch (error) {
    console.error('請求失敗', error);
  }
}
const blockData = ref([
  {
    name: 'test',
    age: 25,
    editIng: false,
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
    label: '儲存',
    icon: 'save',
    status: 'save',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const tempData = ref({
  name: '',
  age: '',
});
// 刪除某列
async function deleteRow(data: eachRowDataType): Promise<void> {
  try {
    const response = await axios.delete(
      `https://demo.mercuryfire.com.tw:49110/crudTest/${data.id}`
    );
    if (response.status === 200) {
      await searchData();
    }
  } catch (error) {
    console.error('請求失敗', error);
  }
}
function editRow(data: eachRowDataType): void {
  try {
    data.editIng = true;
  } catch (error) {
    console.error('請求失敗', error);
  }
}
async function saveRow(data: eachRowDataType): Promise<void> {
  if (!validateNameAndAge(data)) {
    return;
  }
  const response = await axios.patch(
    'https://demo.mercuryfire.com.tw:49110/crudTest',
    {
      name: data.name,
      age: +data.age,
      creatorId: data.creatorId,
      id: data.id,
    }
  );
  if (response.status === 200) {
    await searchData();
  }
}
function handleClickOption<T extends btnType, K extends eachRowDataType>(
  btn: T,
  data: K
) {
  if (!btn || typeof btn.status !== 'string') {
    throw new Error('Invalid button object');
  }

  // 确保传入的数据具有预期的属性
  if (!data || typeof data !== 'object' || !('id' in data)) {
    throw new Error('Invalid data object');
  }

  // 根据按钮状态执行相应操作
  switch (btn.status) {
    case 'delete':
      deleteRow(data);
      break;
    case 'edit':
      editRow(data);
      break;
    case 'save':
      saveRow(data);
      break;
    default:
      throw new Error(`Unsupported button status: ${btn.status}`);
  }
}

async function addNewRow() {
  if (!validateNameAndAge(tempData.value)) {
    return;
  }
  const response = await axios.post(
    'https://demo.mercuryfire.com.tw:49110/crudTest',
    {
      age: +tempData.value.age,
      name: tempData.value.name,
    }
  );
  if (response.status === 200) {
    await searchData();
  }
}
const requiredRule = (val) => !!val || '此欄位不得為空';
const positiveIntegerRule = (val: number) =>
  /^[1-9]\d*$/.test(val) || '請輸入正整數';
function validateNameAndAge(toValidateData: eachRowDataType) {
  const ageValidation = positiveIntegerRule(+toValidateData.age);
  const nameValidation = requiredRule(toValidateData.name);
  if (ageValidation !== true || nameValidation !== true) {
    return false;
  } else {
    return true;
  }
}
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
