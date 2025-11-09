<template>
  <view class="container">
    <view class="user-info">
      <view class="avatar-section" v-if="isLogin">
        <image class="avatar" :src="userInfo.avatarUrl || '../../static/default_avatar.png'" mode="aspectFill"></image>
        <text class="nickname">{{ userInfo.nickName }}</text>
      </view>
      
      <view class="login-section" v-else>
        <image class="default-avatar" src="../../static/default_avatar.png" mode="aspectFill"></image>
        <text class="login-tip">请登录</text>
      </view>
    </view>
    
    <view class="actions">
      <button v-if="!isLogin" class="login-btn" @click="handleLogin">微信登录</button>
      <button v-else class="logout-btn" @click="handleLogout">退出登录</button>
    </view>
    
    <view class="user-menu">
      <view class="menu-item">
        <text class="menu-text">个人资料</text>
        <text class="arrow">></text>
      </view>
      <view class="menu-item">
        <text class="menu-text">设置</text>
        <text class="arrow">></text>
      </view>
      <view class="menu-item">
        <text class="menu-text">关于我们</text>
        <text class="arrow">></text>
      </view>
    </view>
  </view>
</template>

<script>
import { ref, reactive, onMounted } from 'vue'

export default {
  name: 'UserPage',
  setup() {
    const isLogin = ref(false)
    const userInfo = reactive({
      avatarUrl: '',
      nickName: ''
    })
    
    // 页面加载时检查登录状态
    onMounted(() => {
      checkLoginStatus()
    })
    
    // 检查登录状态
    const checkLoginStatus = () => {
      try {
        const loginStatus = uni.getStorageSync('isLogin')
        const storedUserInfo = uni.getStorageSync('userInfo')
        
        if (loginStatus && storedUserInfo) {
          isLogin.value = true
          userInfo.avatarUrl = storedUserInfo.avatarUrl || ''
          userInfo.nickName = storedUserInfo.nickName || ''
        }
      } catch (e) {
        console.error('检查登录状态失败:', e)
      }
    }
    
    // 处理登录
    const handleLogin = () => {
      // #ifdef MP-WEIXIN
      // 微信小程序登录
      wx.login({
        success: function(loginRes) {
          // 正确的微信登录流程应该是将code发送到后端服务器
          // 后端服务器调用auth.code2Session接口换取openid和session_key
          console.log('微信登录获取的code:', loginRes.code)
          
          // 模拟将code发送到后端服务器
          // 实际项目中应该替换为真实的服务器接口
          sendCodeToServer(loginRes.code).then(res => {
            // 假设服务器返回了用户标识和登录状态
            console.log('服务器返回结果:', res)
            
            // 获取用户信息
            wx.getUserProfile({
              desc: '用于完善用户资料',
              success: function(profileRes) {
                isLogin.value = true
                userInfo.avatarUrl = profileRes.userInfo.avatarUrl
                userInfo.nickName = profileRes.userInfo.nickName
                
                // 存储用户信息和登录状态
                try {
                  uni.setStorageSync('isLogin', true)
                  uni.setStorageSync('userInfo', profileRes.userInfo)
                  uni.setStorageSync('openid', res.openid || '') // 存储openid
                  uni.showToast({
                    title: '登录成功',
                    icon: 'success'
                  })
                } catch (e) {
                  console.error('存储用户信息失败:', e)
                  uni.showToast({
                    title: '登录信息存储失败 wx登录',
                    icon: 'none'
                  })
                }
              },
              fail: function(err) {
                console.error('获取用户信息失败:', err)
                uni.showToast({
                  title: '获取用户信息失败',
                  icon: 'none'
                })
              }
            })
          }).catch(err => {
            console.error('服务器处理code失败:', err)
            uni.showToast({
              title: '登录失败',
              icon: 'none'
            })
          })
        },
        fail: function(err) {
          console.error('登录失败:', err)
          uni.showToast({
            title: '登录失败',
            icon: 'none'
          })
        }
      })
      // #endif
      
      // #ifndef MP-WEIXIN
      // 非微信小程序环境，模拟登录
      isLogin.value = true
      userInfo.avatarUrl = '../../static/default_avatar.png'
      userInfo.nickName = '测试用户'
      
      try {
        uni.setStorageSync('isLogin', true)
        uni.setStorageSync('userInfo', {
          avatarUrl: '../../static/default_avatar.png',
          nickName: '测试用户'
        })
        uni.showToast({
          title: '模拟登录成功',
          icon: 'success'
        })
      } catch (e) {
        console.error('模拟登录信息存储失败:', e)
      }
      // #endif
    }
    
    // 模拟将code发送到后端服务器
    const sendCodeToServer = (code) => {
      return new Promise((resolve, reject) => {
        // 实际项目中应该使用uni.request发送请求到后端
        // 示例:
        /*
        uni.request({
          url: 'https://your-domain.com/api/login',
          method: 'POST',
          data: {
            code: code
          },
          success: (res) => {
            resolve(res.data)
          },
          fail: (err) => {
            reject(err)
          }
        })
        */
        
        // 模拟服务器响应
        setTimeout(() => {
          // 模拟服务器通过code调用auth.code2Session得到的结果
          resolve({
            openid: 'user_openid_' + Math.random().toString(36).substr(2, 9),
            session_key: 'session_key_' + Math.random().toString(36).substr(2, 9),
            success: true
          })
        }, 1000)
      })
    }
    
    // 处理退出登录
    const handleLogout = () => {
      uni.showModal({
        title: '提示',
        content: '确定要退出登录吗？',
        success: function(res) {
          if (res.confirm) {
            isLogin.value = false
            userInfo.avatarUrl = ''
            userInfo.nickName = ''
            
            try {
              uni.removeStorageSync('isLogin')
              uni.removeStorageSync('userInfo')
              uni.removeStorageSync('openid')
              uni.showToast({
                title: '已退出登录',
                icon: 'success'
              })
            } catch (e) {
              console.error('清除登录信息失败:', e)
            }
          }
        }
      })
    }
    
    return {
      isLogin,
      userInfo,
      handleLogin,
      handleLogout
    }
  }
}
</script>

<style lang="scss" scoped>
.container {
  min-height: 100vh;
  background-color: #f5f5f5;
}

.user-info {
  background-color: #fff;
  padding: 40rpx;
  margin-bottom: 20rpx;
}

.avatar-section {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.avatar {
  width: 120rpx;
  height: 120rpx;
  border-radius: 50%;
  margin-bottom: 20rpx;
}

.default-avatar {
  width: 120rpx;
  height: 120rpx;
  border-radius: 50%;
  margin-bottom: 20rpx;
}

.nickname {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
}

.login-section {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.login-tip {
  font-size: 28rpx;
  color: #999;
}

.actions {
  background-color: #fff;
  padding: 40rpx;
  margin-bottom: 20rpx;
}

.login-btn, .logout-btn {
  width: 100%;
  height: 80rpx;
  line-height: 80rpx;
  background-color: #07c160;
  color: #fff;
  border-radius: 10rpx;
  font-size: 30rpx;
  border: none;
}

.logout-btn {
  background-color: #fa5151;
}

.user-menu {
  background-color: #fff;
}

.menu-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 30rpx 40rpx;
  border-bottom: 1rpx solid #eee;
}

.menu-item:last-child {
  border-bottom: none;
}

.menu-text {
  font-size: 28rpx;
  color: #333;
}

.arrow {
  color: #999;
}
</style>