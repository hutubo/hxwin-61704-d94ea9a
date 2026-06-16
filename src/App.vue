<script setup lang="ts">
import { computed, reactive, ref } from "vue";

interface ReagentRecord {
  name: string;
  casNo: string;
  batchNo: string;
  storageTemp: string;
  hazardClass: string;
  remaining: string;
  receiveDate: string;
  originalExpiry: string;
  openDate?: string;
  validDaysAfterOpen?: number;
}

interface FormState {
  name: string;
  casNo: string;
  batchNo: string;
  storageTemp: string;
  hazardClass: string;
  receiveDate: string;
  originalExpiry: string;
  openDate: string;
  validDaysAfterOpen: string;
  remaining: string;
}

interface FormErrors {
  name?: string;
  casNo?: string;
  batchNo?: string;
  storageTemp?: string;
  hazardClass?: string;
  receiveDate?: string;
  originalExpiry?: string;
  openDate?: string;
  validDaysAfterOpen?: string;
  remaining?: string;
  general?: string;
}

const project = {
  "id": "hxwin-61704",
  "port": 61704,
  "framework": "vue",
  "title": "实验室试剂有效期管理",
  "subtitle": "试剂批号、储存条件、开封后有效期与领用台账。",
  "stack": "Vue3 + Vite + TypeScript",
  "accent": "#516c92",
  "filters": [
    "全部",
    "即将过期",
    "已过期",
    "低温储存",
    "危险品"
  ],
  "metrics": [
    [
      "在库试剂",
      "146瓶"
    ],
    [
      "30天内过期",
      "11瓶"
    ],
    [
      "低温储存",
      "42瓶"
    ],
    [
      "待废弃",
      "5瓶"
    ]
  ],
  "fields": [
    "试剂名称",
    "CAS号",
    "批号",
    "储存温度",
    "危险分类",
    "剩余量"
  ],
  "records": [
    [
      "乙腈",
      "75-05-8",
      "ACN2405",
      "2-8°C",
      "易燃",
      "1.2L"
    ],
    [
      "胰蛋白酶",
      "9002-07-7",
      "TRY2403",
      "-20°C",
      "生物试剂",
      "8支"
    ],
    [
      "盐酸",
      "7647-01-0",
      "HCL2401",
      "常温",
      "腐蚀性",
      "500mL"
    ]
  ],
  "details": [
    [
      "有效期规则",
      "开封日期叠加开封后有效期，自动识别逾期和临期状态。"
    ],
    [
      "领用记录",
      "每次领用记录人员、用量、剩余量和用途，形成完整台账。"
    ],
    [
      "风险分类",
      "易制毒、易制爆、低温和腐蚀性试剂在首页突出显示。"
    ]
  ],
  "chart": [
    55,
    28,
    72,
    44,
    63
  ],
  "form": [
    "试剂名称",
    "批号",
    "开封日期",
    "剩余量"
  ]
} as const;

const reagentList = ref<ReagentRecord[]>([
  { name: "乙腈", casNo: "75-05-8", batchNo: "ACN2405", storageTemp: "2-8°C", hazardClass: "易燃", remaining: "1.2L", receiveDate: "2024-05-10", originalExpiry: "2026-05-10" },
  { name: "胰蛋白酶", casNo: "9002-07-7", batchNo: "TRY2403", storageTemp: "-20°C", hazardClass: "生物试剂", remaining: "8支", receiveDate: "2024-03-15", originalExpiry: "2025-03-15" },
  { name: "盐酸", casNo: "7647-01-0", batchNo: "HCL2401", storageTemp: "常温", hazardClass: "腐蚀性", remaining: "500mL", receiveDate: "2024-01-20", originalExpiry: "2026-01-20" }
]);

const filter = ref<string>(project.filters[0]);
const selected = ref(0);

const visibleRecords = computed(() => {
  const baseRecords = reagentList.value.map(r => [r.name, r.casNo, r.batchNo, r.storageTemp, r.hazardClass, r.remaining] as string[]);
  if (filter.value === project.filters[0]) return baseRecords;
  return baseRecords.filter((row) => row.join(" ").includes(filter.value));
});
const rows = computed(() => visibleRecords.value.length ? visibleRecords.value : reagentList.value.map(r => [r.name, r.casNo, r.batchNo, r.storageTemp, r.hazardClass, r.remaining] as string[]));

const batchForm = reactive<FormState>({
  name: "",
  casNo: "",
  batchNo: "",
  storageTemp: "",
  hazardClass: "",
  receiveDate: "",
  originalExpiry: "",
  openDate: "",
  validDaysAfterOpen: "",
  remaining: ""
});

const formErrors = reactive<FormErrors>({});
const showBatchForm = ref(true);

const hazardOptions = ["易燃", "腐蚀性", "毒性", "氧化性", "生物试剂", "易制毒", "易制爆", "普通"];
const tempOptions = ["常温", "2-8°C", "-20°C", "-80°C", "0-4°C", "15-25°C"];

const validateRemaining = (value: string): boolean => {
  return /^\d+(\.\d+)?\s*[a-zA-Z\u4e00-\u9fa5]+$/.test(value.trim());
};

const validateForm = (): boolean => {
  Object.keys(formErrors).forEach(key => delete formErrors[key as keyof FormErrors]);
  let isValid = true;

  if (!batchForm.name.trim()) {
    formErrors.name = "试剂名称为必填项";
    isValid = false;
  }
  if (!batchForm.casNo.trim()) {
    formErrors.casNo = "CAS号为必填项";
    isValid = false;
  }
  if (!batchForm.batchNo.trim()) {
    formErrors.batchNo = "批号为必填项";
    isValid = false;
  }
  if (!batchForm.storageTemp.trim()) {
    formErrors.storageTemp = "储存温度为必填项";
    isValid = false;
  }
  if (!batchForm.hazardClass.trim()) {
    formErrors.hazardClass = "危险分类为必填项";
    isValid = false;
  }
  if (!batchForm.receiveDate) {
    formErrors.receiveDate = "入库日期为必填项";
    isValid = false;
  }
  if (!batchForm.originalExpiry) {
    formErrors.originalExpiry = "原始有效期为必填项";
    isValid = false;
  }
  if (!batchForm.remaining.trim()) {
    formErrors.remaining = "剩余量为必填项";
    isValid = false;
  } else if (!validateRemaining(batchForm.remaining)) {
    formErrors.remaining = "剩余量格式不正确，例如：500mL、1.2L、10支";
    isValid = false;
  }

  if (batchForm.receiveDate && batchForm.originalExpiry) {
    const receiveDate = new Date(batchForm.receiveDate);
    const expiryDate = new Date(batchForm.originalExpiry);
    if (expiryDate <= receiveDate) {
      formErrors.originalExpiry = "原始有效期必须晚于入库日期";
      isValid = false;
    }
  }

  if (batchForm.openDate) {
    const openDate = new Date(batchForm.openDate);
    const receiveDate = new Date(batchForm.receiveDate);
    const expiryDate = new Date(batchForm.originalExpiry);
    
    if (batchForm.receiveDate && openDate < receiveDate) {
      formErrors.openDate = "开封日期不能早于入库日期";
      isValid = false;
    }
    if (batchForm.originalExpiry && openDate > expiryDate) {
      formErrors.openDate = "开封日期不能晚于原始有效期";
      isValid = false;
    }
    if (!batchForm.validDaysAfterOpen) {
      formErrors.validDaysAfterOpen = "已填写开封日期时，开封后有效天数为必填项";
      isValid = false;
    } else if (Number(batchForm.validDaysAfterOpen) <= 0 || !Number.isInteger(Number(batchForm.validDaysAfterOpen))) {
      formErrors.validDaysAfterOpen = "请输入有效的正整数天数";
      isValid = false;
    }
  }

  return isValid;
};

const handleBatchSubmit = () => {
  if (!validateForm()) return;

  const newRecord: ReagentRecord = {
    name: batchForm.name.trim(),
    casNo: batchForm.casNo.trim(),
    batchNo: batchForm.batchNo.trim(),
    storageTemp: batchForm.storageTemp.trim(),
    hazardClass: batchForm.hazardClass.trim(),
    remaining: batchForm.remaining.trim(),
    receiveDate: batchForm.receiveDate,
    originalExpiry: batchForm.originalExpiry
  };

  if (batchForm.openDate) {
    newRecord.openDate = batchForm.openDate;
    newRecord.validDaysAfterOpen = parseInt(batchForm.validDaysAfterOpen, 10);
  }

  reagentList.value.unshift(newRecord);
  resetBatchForm();
};

const resetBatchForm = () => {
  batchForm.name = "";
  batchForm.casNo = "";
  batchForm.batchNo = "";
  batchForm.storageTemp = "";
  batchForm.hazardClass = "";
  batchForm.receiveDate = "";
  batchForm.originalExpiry = "";
  batchForm.openDate = "";
  batchForm.validDaysAfterOpen = "";
  batchForm.remaining = "";
  Object.keys(formErrors).forEach(key => delete formErrors[key as keyof FormErrors]);
};
</script>

<template>
  <main class="app-shell" :style="{ '--accent': project.accent }">
    <header class="topbar">
      <div>
        <p class="eyebrow">{{ project.stack }}</p>
        <h1>{{ project.title }}</h1>
        <p class="subtitle">{{ project.subtitle }}</p>
      </div>
      <div class="port-card">
        <span>本地开发端口</span>
        <strong>{{ project.port }}</strong>
        <span>{{ project.id }}</span>
      </div>
    </header>

    <section class="workspace">
      <div>
        <div class="metric-grid">
          <article class="metric" v-for="item in project.metrics" :key="item[0]">
            <span>{{ item[0] }}</span>
            <strong>{{ item[1] }}</strong>
          </article>
        </div>

        <section class="panel">
          <h2>专业记录列表</h2>
          <div class="filters">
            <button v-for="item in project.filters" :key="item" :class="['chip', { active: filter === item }]" @click="filter = item">
              {{ item }}
            </button>
          </div>
          <table class="data-table">
            <thead>
              <tr><th v-for="field in project.fields" :key="field">{{ field }}</th></tr>
            </thead>
            <tbody>
              <tr v-for="(row, index) in rows" :key="row.join('-')" :class="{ selected: selected === index }" @click="selected = index">
                <td v-for="cell in row" :key="cell">{{ cell }}</td>
              </tr>
            </tbody>
          </table>
        </section>

        <section class="panel">
          <h2>指标趋势</h2>
          <div class="chart" aria-label="指标趋势图">
            <div class="bar" v-for="(value, index) in project.chart" :key="index" :style="{ height: value + '%' }">
              <span>{{ value }}</span>
            </div>
          </div>
          <p class="mini-note">图表使用前端mock数据展示关键指标变化，适合作为后续接口联调的UI骨架。</p>
        </section>
      </div>

      <aside>
        <section class="panel">
          <h2>详情工作区</h2>
          <div class="detail-list">
            <div class="detail-item" v-for="item in project.details" :key="item[0]">
              <strong>{{ item[0] }}</strong>
              <p>{{ item[1] }}</p>
            </div>
          </div>
        </section>

        <section class="panel batch-panel">
          <h2>
            <span class="batch-title">试剂批次新增</span>
            <span class="collapse-btn" @click="showBatchForm = !showBatchForm">
              {{ showBatchForm ? '收起' : '展开' }}
            </span>
          </h2>
          <div v-show="showBatchForm" class="batch-form">
            <div class="form-grid form-grid-dense">
              <label :class="{ 'has-error': formErrors.name }">
                试剂名称 <span class="required">*</span>
                <input v-model="batchForm.name" placeholder="如：乙腈" />
                <span v-if="formErrors.name" class="error-text">{{ formErrors.name }}</span>
              </label>
              <label :class="{ 'has-error': formErrors.casNo }">
                CAS号 <span class="required">*</span>
                <input v-model="batchForm.casNo" placeholder="如：75-05-8" />
                <span v-if="formErrors.casNo" class="error-text">{{ formErrors.casNo }}</span>
              </label>
              <label :class="{ 'has-error': formErrors.batchNo }">
                批号 <span class="required">*</span>
                <input v-model="batchForm.batchNo" placeholder="如：ACN2405" />
                <span v-if="formErrors.batchNo" class="error-text">{{ formErrors.batchNo }}</span>
              </label>
              <label :class="{ 'has-error': formErrors.storageTemp }">
                储存温度 <span class="required">*</span>
                <select v-model="batchForm.storageTemp">
                  <option value="">请选择</option>
                  <option v-for="opt in tempOptions" :key="opt" :value="opt">{{ opt }}</option>
                </select>
                <span v-if="formErrors.storageTemp" class="error-text">{{ formErrors.storageTemp }}</span>
              </label>
              <label :class="{ 'has-error': formErrors.hazardClass }">
                危险分类 <span class="required">*</span>
                <select v-model="batchForm.hazardClass">
                  <option value="">请选择</option>
                  <option v-for="opt in hazardOptions" :key="opt" :value="opt">{{ opt }}</option>
                </select>
                <span v-if="formErrors.hazardClass" class="error-text">{{ formErrors.hazardClass }}</span>
              </label>
              <label :class="{ 'has-error': formErrors.remaining }">
                剩余量 <span class="required">*</span>
                <input v-model="batchForm.remaining" placeholder="如：500mL、10支" />
                <span v-if="formErrors.remaining" class="error-text">{{ formErrors.remaining }}</span>
              </label>
              <label :class="{ 'has-error': formErrors.receiveDate }">
                入库日期 <span class="required">*</span>
                <input type="date" v-model="batchForm.receiveDate" />
                <span v-if="formErrors.receiveDate" class="error-text">{{ formErrors.receiveDate }}</span>
              </label>
              <label :class="{ 'has-error': formErrors.originalExpiry }">
                原始有效期 <span class="required">*</span>
                <input type="date" v-model="batchForm.originalExpiry" />
                <span v-if="formErrors.originalExpiry" class="error-text">{{ formErrors.originalExpiry }}</span>
              </label>
              <label :class="{ 'has-error': formErrors.openDate }">
                开封日期
                <input type="date" v-model="batchForm.openDate" />
                <span v-if="formErrors.openDate" class="error-text">{{ formErrors.openDate }}</span>
              </label>
              <label :class="{ 'has-error': formErrors.validDaysAfterOpen }">
                开封后有效天数
                <input v-model="batchForm.validDaysAfterOpen" type="number" min="1" placeholder="如：30" />
                <span v-if="formErrors.validDaysAfterOpen" class="error-text">{{ formErrors.validDaysAfterOpen }}</span>
              </label>
            </div>
            <div class="actions">
              <button class="primary" @click="handleBatchSubmit">提交批次</button>
              <button class="secondary" @click="resetBatchForm">重置</button>
            </div>
          </div>
        </section>

        <section class="panel">
          <h2>快速录入</h2>
          <div class="form-grid">
            <label v-for="field in project.form" :key="field">{{ field }}<input :placeholder="'填写' + field" /></label>
            <textarea placeholder="补充专业备注"></textarea>
          </div>
          <div class="actions">
            <button class="primary">保存记录</button>
            <button class="secondary">导出JSON</button>
          </div>
        </section>
      </aside>
    </section>
  </main>
</template>
