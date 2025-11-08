<template>
  <view class="page-wrapper">
    <view class="task-list-page">
      <!-- 顶部标签页 -->
      <view class="segmented-control-container">
        <uni-segmented-control
          :current="currentTab"
          :values="tabs"
          @clickItem="onClickTab"
          style-type="button"
          active-color="#409eff"
        ></uni-segmented-control>
      </view>

      <!-- 内容区域 -->
      <swiper :current="currentTab" class="swiper-container" @change="onSwiperChange">
        <!-- 待处理 -->
        <swiper-item>
          <scroll-view scroll-y class="task-scroll-view">
            <view v-if="pendingTasks.length > 0" class="task-list">
              <view
                v-for="(task, index) in pendingTasks"
                :key="task.id"
                class="task-item"
                @click="onItemClick(task, index)"
              >
                <view :class="['priority-bar', `priority-${task.priority}`]"></view>
                <view class="checkbox">
                  <view class="checkbox-inner"></view>
                </view>
                <view class="task-content">
                  <view v-if="editingTaskId === task.id" class="edit-title-wrapper" @click.stop>
                    <input
                      class="edit-title-input"
                      type="text"
                      v-model="tempTitle"
                      placeholder="请输入任务标题"
                      :focus="true"
                      confirm-type="done"
                      @confirm="saveEditing"
                      @blur="saveEditing"
                    />
                  </view>
                  <template v-else>
                    <text class="task-title">{{ task.title || '未命名任务' }}</text>
                    <view class="task-meta">
                      <view v-if="task.ddl" class="meta-item">
                        <uni-icons type="calendar" size="14" color="#909399"></uni-icons>
                        <text class="meta-text">{{ formatDate(task.ddl) }}</text>
                      </view>
                      <view v-if="task.duration" class="meta-item">
                        <uni-icons type="pyq" size="14" color="#909399"></uni-icons>
                        <text class="meta-text">{{ formatDuration(task.duration) }}</text>
                      </view>
                    </view>
                  </template>
                </view>
              </view>
            </view>
            <view v-else class="empty-state">
              <image src="/static/empty-box.svg" class="empty-image"></image>
              <text class="empty-text">太棒了！没有待处理的任务！</text>
            </view>
          </scroll-view>
        </swiper-item>

        <!-- 已完成 -->
        <swiper-item>
          <scroll-view scroll-y class="task-scroll-view">
            <view v-if="completedTasks.length > 0" class="task-list">
              <view
                v-for="(task, index) in completedTasks"
                :key="task.id"
                class="task-item completed"
                @click="toggleComplete(task, index)"
              >
                <view :class="['priority-bar', `priority-${task.priority}`]"></view>
                <view class="checkbox">
                  <view class="checkbox-inner">
                    <uni-icons type="checkmarkempty" size="12" color="#fff"></uni-icons>
                  </view>
                </view>
                <view class="task-content">
                  <text class="task-title">{{ task.title }}</text>
                </view>
              </view>
            </view>
            <view v-else class="empty-state">
              <image src="/static/empty-box.svg" class="empty-image"></image>
              <text class="empty-text">还没有已完成的任务哦~</text>
            </view>
          </scroll-view>
        </swiper-item>
      </swiper>
    </view>

    <!-- 自定义悬浮操作按钮 -->
    <view class="custom-fab" @click="onFabClick">
      <uni-icons type="plusempty" size="24" color="#fff"></uni-icons>
    </view>
  </view>
</template>

<script setup>
import { ref } from 'vue';
import { onShow, onUnload } from '@dcloudio/uni-app';

// --- 状态管理 ---
const currentTab = ref(0);
const tabs = ['待处理', '已完成'];

const pendingTasks = ref([]);
const completedTasks = ref([]);
// 内联编辑状态
const editingTaskId = ref(null);
const tempTitle = ref('');

// --- 数据加载函数 ---
const loadTasks = () => {
  const allTasks = uni.getStorageSync('allTasks') || [];
  // 将无截止日期的任务排到后面；同截止日期按优先级与标题排序
  pendingTasks.value = allTasks
    .filter(t => !t.completed)
    .sort((a, b) => {
      const ta = a.ddl ? new Date(a.ddl).getTime() : Number.POSITIVE_INFINITY;
      const tb = b.ddl ? new Date(b.ddl).getTime() : Number.POSITIVE_INFINITY;
      const safeTa = Number.isNaN(ta) ? Number.POSITIVE_INFINITY : ta;
      const safeTb = Number.isNaN(tb) ? Number.POSITIVE_INFINITY : tb;
      if (safeTa !== safeTb) return safeTa - safeTb;
      const pOrder = { high: 0, medium: 1, low: 2 };
      const pa = pOrder[a.priority] ?? 1;
      const pb = pOrder[b.priority] ?? 1;
      if (pa !== pb) return pa - pb;
      return (a.title || '').localeCompare(b.title || '');
    });
  completedTasks.value = allTasks.filter(t => t.completed);
};

// --- 生命周期 & 方法 ---
onShow(() => {
  loadTasks(); // 每次页面显示时都重新加载数据，确保数据最新
});

const onClickTab = (e) => {
  if (currentTab.value !== e.currentIndex) {
    currentTab.value = e.currentIndex;
  }
};

const onSwiperChange = (e) => {
  currentTab.value = e.detail.current;
};

const toggleComplete = (taskToToggle) => {
  // 1. 从本地存储获取最新的任务列表
  const allTasks = uni.getStorageSync('allTasks') || [];

  // 2. 找到需要更新状态的任务
  const task = allTasks.find(t => t.id === taskToToggle.id);

  // 3. 如果找到了任务，则更新其状态
  if (task) {
    task.completed = !task.completed;
  }

  // 4. 将更新后的整个任务列表存回本地存储
  uni.setStorageSync('allTasks', allTasks);

  // 5. 重新加载并渲染页面上的任务列表
  loadTasks();
};

// 点击项时：编辑状态不切换完成；否则切换完成
const onItemClick = (task) => {
  if (editingTaskId.value === task.id) return;
  toggleComplete(task);
};

// 保存编辑（回车或失焦）
const saveEditing = () => {
  const id = editingTaskId.value;
  if (!id && id !== 0) return;
  const title = (tempTitle.value || '').trim();
  const allTasks = uni.getStorageSync('allTasks') || [];
  const idx = allTasks.findIndex(t => t.id === id);
  if (idx !== -1) {
    if (title) {
      allTasks[idx].title = title;
    } else {
      // 空标题直接撤回新建任务
      allTasks.splice(idx, 1);
    }
    uni.setStorageSync('allTasks', allTasks);
  }
  editingTaskId.value = null;
  tempTitle.value = '';
  loadTasks();
};

const formatDuration = (minutes) => {
  const h = Math.floor(minutes / 60);
  const m = minutes % 60;
  let result = '';
  if (h > 0) result += `${h}h `;
  if (m > 0) result += `${m}m`;
  return result.trim() || '-';
};

const formatDate = (dateString) => {
  const date = new Date(dateString);
  const now = new Date();
  const isToday = date.toDateString() === now.toDateString();
  const format = isToday ? 'hh:mm' : 'MM-dd hh:mm';
  
  let fmt = format;
  const o = {
    "M+": date.getMonth() + 1,
    "d+": date.getDate(),
    "h+": date.getHours(),
    "m+": date.getMinutes(),
  };
  for (let k in o) {
    if (new RegExp("(" + k + ")").test(fmt)) {
      let str = o[k] + '';
      fmt = fmt.replace(RegExp.$1, (RegExp.$1.length === 1) ? str : (`00${str}`).substr(str.length));
    }
  }
  return fmt;
};

// 悬浮添加：直接插入新任务并进入编辑
const onFabClick = () => {
  const allTasks = uni.getStorageSync('allTasks') || [];
  const newTask = {
    id: Date.now(),
    title: '',
    completed: false,
    priority: 'medium',
    ddl: null,
    duration: null,
    notes: ''
  };
  allTasks.unshift(newTask);
  uni.setStorageSync('allTasks', allTasks);
  loadTasks();
  editingTaskId.value = newTask.id;
  tempTitle.value = '';
};
</script>

<style lang="scss" scoped>
.page-wrapper {
  position: relative; /* 设置相对定位，为FAB提供定位上下文 */
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.task-list-page {
  display: flex;
  flex-direction: column;
  flex: 1;
  background-color: #f7f8fa;
  /* 移除隐藏，避免遮挡悬浮按钮 */
  overflow: visible;
}

.segmented-control-container {
  padding: 10px 15px;
  background-color: #fff;
  border-bottom: 1px solid #ebeef5;
}

.swiper-container {
  flex: 1;
}

.task-scroll-view {
  height: 100%;
}

.task-list {
  padding: 12px 15px;
}

.task-item {
  display: flex;
  align-items: center;
  background-color: #fff;
  border-radius: 8px;
  margin-bottom: 12px;
  padding: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;

  .priority-bar {
    position: absolute;
    left: 0;
    top: 0;
    bottom: 0;
    width: 4px;
    &.priority-high { background-color: #f56c6c; }
    &.priority-medium { background-color: #e6a23c; }
    &.priority-low { background-color: #409eff; }
  }

  .checkbox {
    width: 22px;
    height: 22px;
    border-radius: 50%;
    border: 2px solid #dcdfe6;
    margin-right: 12px;
    display: flex;
    justify-content: center;
    align-items: center;
    transition: all 0.3s ease;
    flex-shrink: 0;
    .checkbox-inner {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      display: flex;
      justify-content: center;
      align-items: center;
      transition: all 0.3s ease;
    }
  }

  .task-content {
    flex: 1;
    min-width: 0;
    .task-title {
      font-size: 15px;
      color: #303133;
      font-weight: 500;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      transition: color 0.3s ease;
    }
    .task-meta {
      display: flex;
      align-items: center;
      font-size: 12px;
      color: #909399;
      margin-top: 6px;
      .meta-item {
        display: flex;
        align-items: center;
        margin-right: 12px;
        .meta-text {
          margin-left: 4px;
        }
      }
    }
  }

  &.completed {
    .checkbox {
      border-color: #c0c4cc;
      .checkbox-inner {
        background-color: #c0c4cc;
      }
    }
    .task-title {
      color: #909399;
      text-decoration: line-through;
    }
  }
}

.empty-state {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 60vh;
  .empty-image {
    width: 120px;
    height: 120px;
    margin-bottom: 15px;
  }
  .empty-text {
    color: #999;
    font-size: 14px;
  }
}

/* 自定义悬浮按钮样式 */
.custom-fab {
  position: fixed;
  right: 20px;
  bottom: calc(var(--window-bottom) + 20px); /* 适配安全区域 */
  width: 50px;
  height: 50px;
  background-color: #409eff;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  z-index: 1000;
  transition: background-color 0.3s;
}

.custom-fab:active {
  background-color: #337ecc;
}

/* 内联编辑输入框样式 */
.edit-title-wrapper {
  padding: 4px 0;
}
.edit-title-input {
  width: 100%;
  padding: 8px 10px;
  font-size: 15px;
  border: 1px solid #dcdfe6;
  border-radius: 6px;
  background-color: #fff;
}
</style>