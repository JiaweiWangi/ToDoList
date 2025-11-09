<template>
  <view class="schedule-page">
    <!-- 周切换器 -->
    <view class="week-switcher">
      <button @click="prevWeek" class="switch-btn">上一周</button>
      <text class="current-week">{{ weekRange }}</text>
      <button @click="nextWeek" class="switch-btn">下一周</button>
    </view>

    <!-- 星期标题 -->
    <view class="week-header">
      <view 
        v-for="(day, index) in weekDays" 
        :key="index" 
        class="day-header"
        :class="{ today: isToday(day.date) }"
      >
        <text class="day-name">{{ day.name }}</text>
        <text class="day-date">{{ formatDate(day.date) }}</text>
      </view>
    </view>

    <!-- 整周日程展示 -->
    <scroll-view scroll-y class="week-container">
      <view class="week-content">
        <view 
          v-for="(day, index) in weekDays" 
          :key="index" 
          class="day-section"
          :class="{ today: isToday(day.date) }"
        >
          <view class="day-title">
            <text class="day-label">{{ day.name }}</text>
            <text v-if="isToday(day.date)" class="today-tag">今天</text>
          </view>
          
          <view class="event-list">
            <view v-if="getEventsForDay(day.date).length > 0">
              <uni-card 
                v-for="event in getEventsForDay(day.date)" 
                :key="event.id" 
                :is-shadow="true" 
                class="event-card"
              >
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
              <text>暂无日程</text>
            </view>
          </view>
        </view>
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
const currentDate = ref(new Date());
const currentStartOfWeek = ref(getStartOfWeek(new Date()));

// 获取一周的开始日期（周一）
function getStartOfWeek(date) {
  const d = new Date(date);
  const day = d.getDay();
  const diff = d.getDate() - day + (day === 0 ? -6 : 1); // 调整周一开始
  return new Date(d.setDate(diff));
}

// 获取一周的结束日期（周日）
function getEndOfWeek(date) {
  const d = new Date(date);
  const day = d.getDay();
  const diff = d.getDate() + (day === 0 ? 0 : 7 - day);
  return new Date(d.setDate(diff));
}

// 计算一周的日期范围文本
const weekRange = computed(() => {
  const start = currentStartOfWeek.value;
  const end = getEndOfWeek(start);
  return `${start.getFullYear()}年${start.getMonth()+1}月${start.getDate()}日 - ${end.getMonth()+1}月${end.getDate()}日`;
});

// 计算一周的每天日期
const weekDays = computed(() => {
  const days = [];
  const start = new Date(currentStartOfWeek.value);
  
  for (let i = 0; i < 7; i++) {
    const date = new Date(start);
    date.setDate(start.getDate() + i);
    
    const weekdays = ['周一', '周二', '周三', '周四', '周五', '周六', '周日'];
    
    days.push({
      name: weekdays[i],
      date: date
    });
  }
  
  return days;
});

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

// --- 方法 ---
const getEventsForDay = (date) => {
  const dayOfWeek = date.getDay() === 0 ? 6 : date.getDay() - 1; // 转换为我们的格式(0=周一)
  const dateStr = formatDate(date);
  
  // 过滤出当天事件并排序
  return allEvents.value
    .filter(event => {
      // 对于默认事件，使用dayOfWeek匹配
      if (defaultEvents.some(de => de.id === event.id)) {
        return event.dayOfWeek === dayOfWeek;
      }
      
      // 对于用户添加的事件，检查是否是这一天的事件
      if (event.date) {
        return event.date === dateStr;
      }
      
      // 如果既没有date也没有是默认事件，则使用dayOfWeek匹配
      return event.dayOfWeek === dayOfWeek;
    })
    .sort((a, b) => a.startTime.localeCompare(b.startTime));
};

// 上一周
const prevWeek = () => {
  const newStart = new Date(currentStartOfWeek.value);
  newStart.setDate(newStart.getDate() - 7);
  currentStartOfWeek.value = newStart;
};

// 下一周
const nextWeek = () => {
  const newStart = new Date(currentStartOfWeek.value);
  newStart.setDate(newStart.getDate() + 7);
  currentStartOfWeek.value = newStart;
};

// 判断是否是今天
const isToday = (date) => {
  const today = new Date();
  return date.toDateString() === today.toDateString();
};

// 格式化日期为YYYY-MM-DD
const formatDate = (date) => {
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');
  return `${year}-${month}-${day}`;
};

const onAddEvent = () => {
  uni.navigateTo({ url: '/pages/schedule/form' });
};

// --- FAB 配置 ---
const fabPattern = {
  color: '#7A7E83',
  backgroundColor: '#fff',
  selectedColor: '#409eff',
  buttonColor: '#409eff',
  icon: 'plusempty',
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

.week-switcher {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  background-color: #fff;
  border-bottom: 1px solid #ebeef5;
  
  .switch-btn {
    padding: 5px 10px;
    background-color: #ecf5ff;
    color: #409eff;
    border: none;
    border-radius: 4px;
    font-size: 14px;
  }
  
  .current-week {
    font-size: 16px;
    font-weight: bold;
    color: #303133;
  }
}

.week-header {
  display: flex;
  background-color: #fff;
  border-bottom: 1px solid #ebeef5;
  
  .day-header {
    flex: 1;
    text-align: center;
    padding: 10px 0;
    
    &.today {
      color: #409eff;
      font-weight: bold;
    }
    
    .day-name {
      display: block;
      font-size: 14px;
    }
    
    .day-date {
      display: block;
      font-size: 12px;
      margin-top: 2px;
      color: #909399;
    }
  }
}

.week-container {
  flex: 1;
}

.week-content {
  padding: 10px;
}

.day-section {
  margin-bottom: 20px;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  background-color: #fff;
  
  &.today {
    border: 1px solid #409eff;
  }
}

.day-title {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 15px;
  background-color: #f5f7fa;
  border-bottom: 1px solid #ebeef5;
  
  .day-label {
    font-size: 16px;
    font-weight: bold;
    color: #303133;
  }
  
  .today-tag {
    font-size: 12px;
    color: #409eff;
    background-color: #ecf5ff;
    padding: 2px 6px;
    border-radius: 4px;
  }
}

.event-list {
  padding: 10px 15px;
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
  padding: 20px 0;
  color: #999;
  font-size: 14px;
}
</style>