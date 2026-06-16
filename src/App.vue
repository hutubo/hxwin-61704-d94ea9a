<script setup lang="ts">
import { computed, ref } from "vue";

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
const filter = ref<string>(project.filters[0]);
const selected = ref(0);
const visibleRecords = computed(() => {
  if (filter.value === project.filters[0]) return project.records;
  return project.records.filter((row) => row.join(" ").includes(filter.value));
});
const rows = computed(() => visibleRecords.value.length ? visibleRecords.value : project.records);
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
