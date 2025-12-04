<template>
  <div class="tdFooter">
    <div class="footer-left">
      <span class="counter">总计:{{ totalCount }} | 未完成:{{ activeCount }} | 已完成:{{ completedCount }}</span>
    </div>
    <div class="footer-center">
      <div class="tdTabs">
        <a 
          :class="{ active: tabType === 0 }" 
          @click="tabClick(0)"
        >全部</a>
        <a 
          :class="{ active: tabType === 1 }" 
          @click="tabClick(1)"
        >未完成</a>
        <a 
          :class="{ active: tabType === 2 }" 
          @click="tabClick(2)"
        >已完成</a>
      </div>
    </div>
    <div class="footer-right">
      <button 
        v-if="completedCount > 0"
        @click="clearCompleted"
        class="clear-btn"
      >
        清除已完成 ({{ completedCount }})
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: ["totalCount", "activeCount", "completedCount", "tabType"],
  methods: {
    tabClick(newTabType) {
      this.$emit("changeTabType", newTabType);
    },
    clearCompleted() {
      this.$emit("clearCompleted");
    }
  }
}
</script>

<style scoped>
.tdFooter {
  background-color: #fff;
  padding: 10px 20px;
  margin: 20px 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-radius: 10px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.footer-left {
  font-size: 14px;
  color: #666;
}

.counter {
  background-color: #f5f5f5;
  padding: 4px 8px;
  border-radius: 4px;
}

.tdTabs {
  display: flex;
  gap: 10px;
}

.tdTabs a {
  padding: 0 10px;
  cursor: pointer;
  color: #666;
  text-decoration: none;
  font-size: 14px;
}

.tdTabs a.active {
  color: red;
  font-weight: bold;
}

.clear-btn {
  padding: 5px 10px;
  background-color: #ff6b6b;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
}
</style>