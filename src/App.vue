<script>
export default {
  onLaunch: function () {
    console.log('App Launch')
  },
  onShow: function () {
    console.log('App Show')
  },
  onHide: function () {
    console.log('App Hide')
  },
}
</script>

<style>
  /* 每个页面公共css */
  .page-wrapper {
    padding-bottom: 50px; /* 预留底部安全距离，避免内容被 TabBar 遮挡 */
  }
</style>
