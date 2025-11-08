<template>
  <view class="task-form-page">
    <view class="page-title">新建任务</view>
    <uni-forms ref="formRef" :modelValue="formData" :rules="rules" class="form-container">
      <!-- 任务标题 -->
      <uni-forms-item label="任务标题" name="title" required>
        <uni-easyinput type="text" v-model="formData.title" placeholder="请输入任务标题" />
      </uni-forms-item>

      <!-- 预估时长 -->
      <uni-forms-item label="预估时长" name="duration">
        <uni-data-select
          v-model="formData.duration"
          :localdata="durationOptions"
          placeholder="请选择预估时长"
        ></uni-data-select>
      </uni-forms-item>

      <!-- 优先级 -->
      <uni-forms-item label="优先级" name="priority">
        <uni-data-select
          v-model="formData.priority"
          :localdata="priorityOptions"
          placeholder="请选择优先级"
        ></uni-data-select>
      </uni-forms-item>

      <!-- 截止日期 -->
      <uni-forms-item label="截止日期" name="ddl">
        <uni-datetime-picker type="datetime" v-model="formData.ddl" />
      </uni-forms-item>

      <!-- 任务备注 -->
      <uni-forms-item label="任务备注" name="notes">
        <uni-easyinput type="textarea" autoHeight v-model="formData.notes" placeholder="请输入备注信息" />
      </uni-forms-item>
    </uni-forms>

    <!-- 底部提交按钮 -->
    <view class="footer-actions">
      <button class="save-button" @click="handleSubmit">保存</button>
    </view>
  </view>
</template>

<script setup>
import { ref, reactive } from 'vue';

// --- 表单实例 ---
const formRef = ref(null);

// --- 表单数据 ---
const formData = reactive({
  title: '',
  duration: 60, // 默认60分钟, 对应1小时
  priority: 'medium',
  ddl: '',
  notes: ''
});

// --- 表单选项 ---
const durationOptions = ref([
  { value: 30, text: '30分钟' },
  { value: 60, text: '1小时' },
  { value: 90, text: '1.5小时' },
  { value: 120, text: '2小时' },
  { value: 180, text: '3小时' },
]);

const priorityOptions = ref([
  { value: 'high', text: '高' },
  { value: 'medium', text: '中' },
  { value: 'low', text: '低' },
]);

// --- 校验规则 ---
const rules = reactive({
  title: {
    rules: [{
      required: true,
      errorMessage: '任务标题不能为空',
    }],
  },
});

// --- 方法 ---
const handleSubmit = () => {
  formRef.value.validate().then(res => {
    console.log('表单校验成功，数据:', formData);

    // 1. 准备新任务数据
    const newTask = {
      id: `task_${Date.now()}`,
      completed: false,
      ...formData
    };

    // 2. 从本地存储获取现有任务列表
    const allTasks = uni.getStorageSync('allTasks') || [];

    // 3. 将新任务添加到列表中
    allTasks.push(newTask);

    // 4. 将更新后的列表存回本地存储
    uni.setStorageSync('allTasks', allTasks);

    uni.showToast({
      title: '保存成功',
      icon: 'success'
    });

    // 5. 直接返回上一页，列表页的 onShow 生命周期会自动刷新
    setTimeout(() => {
      uni.navigateBack();
    }, 800);

  }).catch(err => {
    console.log('表单校验失败:', err);
  });
};
</script>

<style lang="scss" scoped>
.task-form-page {
  background-color: #f4f4f5;
  min-height: 100vh;
  padding-bottom: 80px; // 为底部按钮留出空间
}

.form-container {
  margin: 12px;
  padding: 15px;
  background-color: #fff;
  border-radius: 8px;
}

// 覆盖uni-ui组件样式，使其更美观
::v-deep .uni-forms-item {
  margin-bottom: 18px;
  .uni-forms-item__label {
    font-weight: 500;
    color: #303133;
  }
  .uni-easyinput__content {
    border-radius: 4px;
  }
}

.footer-actions {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: #fff;
  padding: 10px 20px;
  padding-bottom: calc(10px + constant(safe-area-inset-bottom));
  padding-bottom: calc(10px + env(safe-area-inset-bottom));
  border-top: 1px solid #ebeef5;
  box-shadow: 0 -2px 8px rgba(0, 0, 0, 0.04);
}

.save-button {
  background-color: #409eff;
  color: white;
  border-radius: 50px;
  font-size: 16px;
  font-weight: bold;
  height: 44px;
  line-height: 44px;
  &:active {
    background-color: #337ecc;
  }
}
</style>