<template>
  <view class="today-schedule">
    <!-- 顶部日期 -->
    <view class="header">
      <text class="date">{{ todayDate }}</text>
    </view>

    <!-- 时间线容器 -->
    <scroll-view scroll-y class="timeline-container">
      <view class="timeline">
        <!-- 时间刻度 -->
        <view class="time-markers">
          <view v-for="hour in hours" :key="hour" class="time-marker">
            <text class="time-text">{{ hour }}:00</text>
          </view>
        </view>

        <!-- 日程卡片 -->
        <view class="schedule-items-wrapper">
          <view
            v-for="item in scheduleItems"
            :key="item.id"
            class="schedule-item"
            :class="[item.type === 'fixed' ? 'fixed-event' : 'task-event']"
            :style="calculatePosition(item)"
          >
            <view class="card-content">
              <text class="title">{{ item.title }}</text>
              <text class="time">{{ item.startTime }} - {{ item.endTime }}</text>
            </view>
            <view v-if="item.type === 'task' && item.priority === 'high'" class="priority-badge"></view>
          </view>
        </view>
      </view>
    </scroll-view>

    <!-- 底部操作按钮 -->
    <view class="footer">
      <button class="schedule-button" @click="handleSchedule">
        <text>🚀 一键智能安排</text>
      </button>
    </view>
  </view>
</template>

<script setup>
import { ref, reactive, computed } from 'vue';

// --- 响应式数据 ---
const scheduleItems = reactive([
  { id: 'f1', type: 'fixed', title: '高等数学 (课)', startTime: '09:00', endTime: '11:00' },
  { id: 't1', type: 'task', title: '完成高数作业', startTime: '11:15', endTime: '12:30', priority: 'high' },
  { id: 'f2', type: 'fixed', title: '团队周会 (会)', startTime: '14:00', endTime: '15:30' },
  { id: 't2', type: 'task', title: '回复Email', startTime: '15:45', endTime: '16:15', priority: 'low' },
  { id: 't3', type: 'task', title: '背单词', startTime: '08:00', endTime: '08:30', priority: 'low' }
]);

// 时间线小时范围
const hours = Array.from({ length: 15 }, (_, i) => 8 + i); // 8:00 to 22:00

// --- 计算属性 ---
const todayDate = computed(() => {
  const now = new Date();
  const month = now.getMonth() + 1;
  const day = now.getDate();
  const weekday = ['星期日', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六'][now.getDay()];
  return `${month}月${day}日 ${weekday}`;
});

// --- 方法 ---

// 每小时的高度 (px)
const HOUR_HEIGHT = 60;
// 时间线开始的小时
const START_HOUR = 8;

/**
 * 计算日程卡片在时间线上的位置和高度
 * @param {object} item - 日程项
 */
const calculatePosition = (item) => {
  const [startH, startM] = item.startTime.split(':').map(Number);
  const [endH, endM] = item.endTime.split(':').map(Number);

  const top = (startH - START_HOUR + startM / 60) * HOUR_HEIGHT;
  const height = ((endH * 60 + endM) - (startH * 60 + startM)) / 60 * HOUR_HEIGHT;

  return {
    top: `${top}px`,
    height: `${height}px`,
  };
};

/**
 * 点击“一键智能安排”按钮
 */
const handleSchedule = () => {
  console.log('开始智能安排');
  uni.showToast({
    title: '正在为您智能安排...',
    icon: 'none'
  });
};
</script>

<style lang="scss" scoped>
.today-schedule {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background-color: #f7f8fa;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.header {
  padding: 15px 20px;
  background-color: #fff;
  border-bottom: 1px solid #ebedf0;
  text-align: center;
  .date {
    font-size: 18px;
    font-weight: bold;
    color: #333;
  }
}

.timeline-container {
  flex: 1;
  position: relative;
  padding: 0 20px;
}

.timeline {
  position: relative;
  padding-left: 60px; /* 为时间刻度留出空间 */
  padding-top: 20px;
  padding-bottom: 20px;
}

.time-markers {
  position: absolute;
  left: 0;
  top: 20px;
  width: 60px;
  .time-marker {
    height: 60px; /* 每小时的高度 */
    position: relative;
    &:not(:last-child)::after {
      content: '';
      position: absolute;
      left: 45px;
      top: 0;
      width: 1px;
      height: 100%;
      background-color: #e0e0e0;
    }
    .time-text {
      position: absolute;
      top: -8px; /* 向上偏移，对准刻度线 */
      left: 0;
      font-size: 12px;
      color: #999;
    }
  }
}

.schedule-items-wrapper {
  position: relative;
  height: calc(15 * 60px); /* 8am to 10pm is 14 hours, but we need 15 markers for 8 to 22 */
}

.schedule-item {
  position: absolute;
  left: 10px;
  right: 0;
  border-radius: 8px;
  padding: 8px 12px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  transition: all 0.3s ease;
  overflow: hidden;

  .card-content {
    display: flex;
    flex-direction: column;
  }

  .title {
    font-size: 14px;
    font-weight: 600;
    color: #333;
  }

  .time {
    font-size: 12px;
    color: #666;
    margin-top: 4px;
  }
  
  .priority-badge {
    position: absolute;
    top: 8px;
    right: 8px;
    width: 10px;
    height: 10px;
    background-color: #fa3534;
    border-radius: 50%;
  }
}

.fixed-event {
  background-color: #e6f7ff;
  border: 1px solid #91d5ff;
  .title {
    color: #0050b3;
  }
}

.task-event {
  background-color: #fffbe6;
  border: 1px dashed #ffe58f;
  .title {
    color: #ad8b00;
  }
}

.footer {
  padding: 15px 20px;
  background-color: #fff;
  border-top: 1px solid #ebedf0;
  box-shadow: 0 -2px 8px rgba(0, 0, 0, 0.05);
}

.schedule-button {
  background: linear-gradient(45deg, #409eff, #67c23a);
  color: white;
  border: none;
  border-radius: 50px;
  font-size: 16px;
  font-weight: bold;
  height: 44px;
  line-height: 44px;
  text-align: center;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  &:active {
    transform: scale(0.98);
    opacity: 0.9;
  }
}
</style>