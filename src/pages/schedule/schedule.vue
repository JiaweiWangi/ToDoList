<template>
  <view class="schedule-page">
    <!-- 星期切换器 -->
    <view class="day-switcher">
      <uni-segmented-control
        :current="currentDay"
        :values="daysOfWeek"
        @clickItem="onDayChange"
        style-type="text"
        active-color="#409eff"
      ></uni-segmented-control>
    </view>

    <!-- 日程列表 -->
    <scroll-view scroll-y class="event-list-container">
      <view v-if="displayEvents.length > 0" class="event-list">
        <uni-card v-for="event in displayEvents" :key="event.id" :is-shadow="true" class="event-card">
          <text class="event-title">{{ event.title }}</text>
          <view class="event-details">
            <view class="detail-item">
              <uni-icons type="clock" size="16" color="#909399"></uni-icons>
              <text class="detail-text">{{ event.startTime }} - {{ event.endTime }}</text>
            </view>
            <view class="detail-item">
              <uni-icons type="location" size="16" color="#909399"></uni-icons>
              <text class="detail-text">{{ event.location }}</text>
            </view>
          </view>
        </uni-card>
      </view>
      <view v-else class="empty-state">
        <text>今天没有固定日程，好好休息一下吧！</text>
      </view>
    </scroll-view>

    <!-- 添加按钮 -->
    <uni-fab
      :pattern="fabPattern"
      horizontal="right"
      vertical="bottom"
      @fabClick="onAddEvent"
    ></uni-fab>

  </view>
</template>

<script setup>
import { ref, computed } from 'vue';
import { onShow } from '@dcloudio/uni-app';

// --- 状态管理 ---
const currentDay = ref(new Date().getDay() - 1); // 默认为今天, 0=周一, 1=周二...
if (currentDay.value < 0) currentDay.value = 6; // 周日 getDay() 是 0

const daysOfWeek = ['周一', '周二', '周三', '周四', '周五', '周六', '周日'];

// 默认示例日程（可与用户新增日程合并显示）
const defaultEvents = [
  { id: 'f1', title: '高等数学', dayOfWeek: 0, startTime: '09:00', endTime: '11:00', location: 'A101教室' },
  { id: 'f2', title: '大学英语', dayOfWeek: 0, startTime: '14:00', endTime: '16:00', location: 'B203报告厅' },
  { id: 'f3', title: 'Java程序设计', dayOfWeek: 1, startTime: '10:00', endTime: '12:00', location: 'C505机房' },
  { id: 'f4', title: '线性代数', dayOfWeek: 2, startTime: '09:00', endTime: '11:00', location: 'A101教室' },
  { id: 'f5', title: '团队协作项目会议', dayOfWeek: 3, startTime: '15:00', endTime: '16:30', location: '线上会议' },
  { id: 'f6', title: '健身房锻炼', dayOfWeek: 4, startTime: '19:00', endTime: '20:30', location: '学校健身房' },
];

// 所有日程（默认 + 用户新增）
const allEvents = ref([...defaultEvents]);

const loadEvents = () => {
  const saved = uni.getStorageSync('allFixedEvents') || [];
  allEvents.value = [...defaultEvents, ...saved];
};

// --- 计算属性 ---
const displayEvents = computed(() => {
  return allEvents.value
    .filter(event => event.dayOfWeek === currentDay.value)
    .sort((a, b) => a.startTime.localeCompare(b.startTime));
});

// --- FAB 配置 ---
const fabPattern = {
  color: '#7A7E83',
  backgroundColor: '#fff',
  selectedColor: '#409eff',
  buttonColor: '#409eff',
  icon: 'plusempty',
};

// --- 方法 ---
const onDayChange = (e) => {
  currentDay.value = e.currentIndex;
};

const onAddEvent = () => {
  uni.navigateTo({ url: '/pages/schedule/form' });
};

// 每次页面显示时刷新数据，确保新增内容可见
onShow(() => {
  loadEvents();
});

</script>

<style lang="scss" scoped>
.schedule-page {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background-color: #f9f9f9;
}

.day-switcher {
  background-color: #fff;
  border-bottom: 1px solid #ebeef5;
}

.event-list-container {
  flex: 1;
  height: 100%;
}

.event-list {
  padding: 12px;
}

.event-card {
  margin-bottom: 12px;
  .event-title {
    font-size: 16px;
    font-weight: bold;
    color: #303133;
    margin-bottom: 15px;
    display: block;
  }
  .event-details {
    .detail-item {
      display: flex;
      align-items: center;
      font-size: 14px;
      color: #606266;
      &:not(:last-child) {
        margin-bottom: 10px;
      }
      .detail-text {
        margin-left: 8px;
      }
    }
  }
}

.empty-state {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 50vh;
  color: #999;
  font-size: 14px;
}
</style>