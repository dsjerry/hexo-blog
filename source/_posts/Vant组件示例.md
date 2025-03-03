---
title: Vant组件示例
toc: true
date: 2025-03-03 22:51:34
tags: [组件, vue]
categories: 前端学习
thumbnail: https://s21.ax1x.com/2025/03/03/pEGHjr8.png
cover: https://s21.ax1x.com/2025/03/03/pEGHjr8.png
---

📦 复制就能使用？

<!-- more -->

Vant4，[官方文档](https://vant-ui.github.io/vant/#/zh-CN)

# 渲染组件

## 嵌套的collapse

嵌套的折叠面板，建议名字：`NestedCollapse.vue`

```vue
<template>
  <van-collapse class="nested-collapse" v-model="activeNames" accordion>
    <van-collapse-item v-for="item in items" :key="item.id" :name="item.id">
      <template #right-icon>
        <van-icon
          v-if="item.childList?.length"
          :name="activeNames == item.id ? 'arrow-up' : 'arrow-down'"
        />
      </template>
      <template #title>
        <div class="title" @click.stop="onItemClick(item)">
          {{ item.name }}
        </div>
      </template>
      <nested-collapse
        v-if="item.childList?.length"
        :items="item.childList"
        @click="(item) => onItemClick(item)"
      />
    </van-collapse-item>
  </van-collapse>
</template>

<script setup lang="ts">
/**
 * 嵌套折叠面板，每层传递childList
 */

interface Item {
  id: string | number;
  name: string;
  childList?: Item[];
}

// 手风琴模式（accordion）绑定普通类型即可，否则需要绑定数组
const props = defineProps<{
  items: Item[];
}>();

const emit = defineEmits<{
  (e: "click", item: Item): void;
}>();

const activeNames = ref("");

function onItemClick(item: Item) {
  emit("click", item);
}
</script>

<style lang="scss" scoped>
.nested-collapse {
  --van-cell-horizontal-padding: 4px;
  --van-cell-group-background: transparent;
  --van-cell-background: transparent;

  :deep(.van-collapse-item__content) {
    --van-collapse-item-content-background: transparent;
    padding-left: 16px;
  }
  // 留出点空位
  // 点击标题时阻止了组件的默认事件，如果宽度为100%会导致右边的箭头无法点击
  .title {
    width: 96%;
  }
}
</style>
```

## 单行日期选择

当天选中为主题色作为背景色，选中其他日期主题色为边框，建议名称：`InlineDatePicker.vue`

```vue
<template>
  <div class="calendar">
    <!-- 周标题 -->
    <div v-if="!inSameRound" class="weekdays">
      <div v-for="(day, index) in weekdays" :key="index" class="weekday">
        {{ day }}
      </div>
    </div>
    <!-- 日期 -->
    <div class="dates">
      <div
        v-for="(date, index) in dates"
        :key="index"
        class="date"
        :class="{ today: date.isToday, active: isSelected(date) }"
        @click="selectDate(date)"
      >
        <span v-if="inSameRound">{{ date.day }}</span>
        <span v-if="date.date">{{ date.date }}</span>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
interface Props {
  inSameRound?: boolean; // 星期和日期在一个圆形内
}
interface Emits {
  (e: "select", date: DateType): void;
}
interface DateType {
  day: string; // 周
  date: number; // 日
  fullDate: string;
  isToday: boolean;
}

const props = defineProps<Props>();
const emit = defineEmits<Emits>();

const weekdays = ref(["日", "一", "二", "三", "四", "五", "六"]);
const today = new Date();

const selectedDate = ref<DateType | null>(null);
const isSelected = (date: DateType) => {
  return selectedDate.value?.fullDate === date.fullDate;
};

// 生成当前周的日期数据
const dates = computed(() => {
  const startOfWeek = new Date(today);
  startOfWeek.setDate(today.getDate() - today.getDay());

  return Array.from({ length: 7 }).map((_, index) => {
    const currentDate = new Date(startOfWeek);
    currentDate.setDate(startOfWeek.getDate() + index);

    return {
      day: weekdays.value[index],
      date: currentDate.getDate(),
      isToday: currentDate.toDateString() === today.toDateString(),
      fullDate: currentDate.toLocaleDateString(),
    };
  });
});

function selectDate(date: DateType) {
  selectedDate.value = date;
  emit("select", date);
}
</script>

<style lang="scss" scoped>
.calendar {
  display: flex;
  flex-direction: column;
  text-align: center;
}

.weekdays,
.dates {
  display: flex;
  justify-content: space-around;
}

.weekday,
.date {
  flex: 1;
}

.date {
  $date-size: 30px;

  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: $date-size;
  max-width: $date-size;
  height: $date-size;
  margin-top: 5px;
  border-radius: 100%;
  box-sizing: border-box;
}

.date.today {
  color: white;
  background-color: var(--van-primary-color);
}

.date.active {
  font-weight: bold;
  border: 1px solid var(--van-primary-color);
}
</style>
```