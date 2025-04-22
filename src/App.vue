<script setup lang="ts">
import { defineAsyncComponent } from 'vue';
const VueTable = defineAsyncComponent(() => import("./components/VueTable.vue"));
</script>

<template>
  <div class="app-container">
    <h1>Vue表格组件</h1>
    <div class="app-description">
      <div class="description-title">这是一个简单的Vue表格组件，支持以下功能：</div>
      <div class="features-wrapper">
        <div class="left-decoration">
          <div class="vertical-line"></div>
        </div>
        <div class="features-list">
          <div class="feature-group">
            <div class="feature-title">通过<strong>右键菜单</strong>可执行以下操作：</div>
            <div class="sub-features">
              <div class="sub-feature">
                <span class="dot"></span>
                <span>新增行/列</span>
              </div>
              <div class="sub-feature">
                <span class="dot"></span>
                <span>删除行/列</span>
              </div>
              <div class="sub-feature">
                <span class="dot"></span>
                <span>复制/粘贴单元格内容</span>
              </div>
              <div class="sub-feature">
                <span class="dot"></span>
                <span>合并/取消合并单元格</span>
              </div>
            </div>
          </div>
          <div class="feature-item">
            <span class="dot"></span>
            <span>支持直接编辑单元格内容（双击单元格）</span>
          </div>
          <div class="feature-item">
            <span class="dot"></span>
            <span>支持编辑表头（双击表头）</span>
          </div>
          <div class="feature-item">
            <span class="dot"></span>
            <span>选择操作：点击单元格进行选择，按住Shift可选择范围，Ctrl/Command+点击可多选</span>
          </div>
        </div>
      </div>
    </div>
    <VueTable />
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html,
body {
  font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB",
    "Microsoft YaHei", "微软雅黑", Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  background-color: #f5f7fa;
}

#app {
  width: 100%;
  min-height: 100vh;
  padding: 20px;
}

.app-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 30px;
  background-color: white;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
}

h1 {
  color: #1890ff;
  text-align: center;
  margin-bottom: 24px;
  font-size: 28px;
  letter-spacing: 0.5px;
  position: relative;
  padding-bottom: 12px;
}

h1:after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 80px;
  height: 3px;
  background-color: #1890ff;
  border-radius: 3px;
}

.app-description {
  background-color: #e6f7ff;
  padding: 20px 40px;
  border-radius: 8px;
  margin-bottom: 30px;
  line-height: 1.7;
  font-size: 15px;
  color: #3c4858;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
}

.description-title {
  font-size: 16px;
  font-weight: 500;
  margin-left: 80px;
  margin-bottom: 15px;
  color: #1a2a3a;
  text-align: left;
}

.features-wrapper {
  display: flex;
  position: relative;
  max-width: 90%;
  margin: 0 auto;
}

.left-decoration {
  width: 40px;
  position: relative;
  flex-shrink: 0;
}

.vertical-line {
  position: absolute;
  left: 20px;
  top: 10px;
  bottom: 10px;
  width: 2px;
  background-color: rgba(24, 144, 255, 0.15);
}

.features-list {
  flex-grow: 1;
  padding-left: 0;
}

.feature-group {
  margin-bottom: 15px;
}

.feature-title {
  position: relative;
  margin-bottom: 8px;
  text-align: left;
  color: #2c3e50;
  font-weight: 500;
}

.sub-features {
  padding-left: 20px;
}

.sub-feature {
  margin-bottom: 4px;
  position: relative;
  display: flex;
  align-items: center;
}

.feature-item {
  margin-bottom: 10px;
  position: relative;
  display: flex;
  align-items: center;
}

.dot {
  display: inline-block;
  width: 6px;
  height: 6px;
  background-color: rgba(24, 144, 255, 0.7);
  border-radius: 50%;
  margin-right: 8px;
  flex-shrink: 0;
}

.app-description strong {
  color: #1890ff;
  font-weight: 600;
}
</style>
