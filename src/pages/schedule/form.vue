<template>
  <view class="schedule-form-page">
    <uni-forms ref="formRef" :modelValue="formData" :rules="rules" class="form-container">
      <!-- 日程标题 -->
      <uni-forms-item label="日程标题" name="title" required>
        <uni-easyinput type="text" v-model="formData.title" placeholder="请输入日程标题" />
      </uni-forms-item>

      <!-- 星期选择 -->
      <uni-forms-item label="星期" name="dayOfWeek" required>
        <uni-data-select
          v-model="formData.dayOfWeek"
          :localdata="weekOptions"
          placeholder="请选择星期"
        ></uni-data-select>
      </uni-forms-item>

      <!-- 开始时间 -->
      <uni-forms-item label="开始时间" name="startTime" required>
        <uni-datetime-picker type="time" v-model="formData.startTime" />
      </uni-forms-item>

      <!-- 结束时间 -->
      <uni-forms-item label="结束时间" name="endTime" required>
        <uni-datetime-picker type="time" v-model="formData.endTime" />
      </uni-forms-item>

      <!-- 地点 -->
      <uni-forms-item label="地点" name="location">
        <uni-easyinput type="text" v-model="formData.location" placeholder="请输入地点(可选)" />
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

// 表单引用
const formRef = ref(null);

// 星期选项：与固定日程页的索引相匹配 (周一=0 ... 周日=6)
const weekOptions = ref([
  { value: 0, text: '周一' },
  { value: 1, text: '周二' },
  { value: 2, text: '周三' },
  { value: 3, text: '周四' },
  { value: 4, text: '周五' },
  { value: 5, text: '周六' },
  { value: 6, text: '周日' },
]);

// 表单数据
const formData = reactive({
  title: '',
  dayOfWeek: 0,
  startTime: '09:00',
  endTime: '10:00',
  location: ''
});

// 校验规则
const rules = reactive({
  title: {
    rules: [{ required: true, errorMessage: '日程标题不能为空' }]
  },
  dayOfWeek: {
    rules: [{ required: true, errorMessage: '请选择星期' }]
  },
  startTime: {
    rules: [{ required: true, errorMessage: '请选择开始时间' }]
  },
  endTime: {
    rules: [
      { required: true, errorMessage: '请选择结束时间' },
      {
        validateFunction: (rule, value, data, callback) => {
          if (!data.startTime || !value) return callback();
          const [sh, sm] = data.startTime.split(':').map(Number);
          const [eh, em] = value.split(':').map(Number);
          const start = sh * 60 + sm;
          const end = eh * 60 + em;
          if (end <= start) {
            callback('结束时间必须晚于开始时间');
          } else {
            callback();
          }
        }
      }
    ]
  }
});

// 保存提交
const handleSubmit = () => {
  formRef.value.validate().then(() => {
    const newEvent = {
      id: `fixed_${Date.now()}`,
      title: formData.title,
      dayOfWeek: formData.dayOfWeek,
      startTime: formData.startTime,
      endTime: formData.endTime,
      location: formData.location
    };

    const allFixedEvents = uni.getStorageSync('allFixedEvents') || [];
    allFixedEvents.push(newEvent);
    uni.setStorageSync('allFixedEvents', allFixedEvents);

    uni.showToast({ title: '已保存', icon: 'success' });
    setTimeout(() => {
      uni.navigateBack();
    }, 400);
  }).catch(() => {
    uni.showToast({ title: '请完善表单信息', icon: 'none' });
  });
};
</script>

<style lang="scss" scoped>
.schedule-form-page {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background-color: #f9f9f9;
}

.form-container {
  background-color: #fff;
  padding: 16px;
}

.footer-actions {
  padding: 16px;
  background-color: #fff;
  border-top: 1px solid #ebedf0;
}

.save-button {
  background-color: #409eff;
  color: #fff;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  height: 44px;
  line-height: 44px;
  text-align: center;
}
</style>