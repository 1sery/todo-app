<template>
    <div class="tdContainer">
        <!-- 空状态提示 -->
        <div v-if="todos.length === 0" class="empty-state">
            <div v-if="searchMode && searchKeyword">
                <div class="empty-title">没有找到匹配的任务</div>
                <div class="empty-subtitle">未找到包含 "<strong>{{ searchKeyword }}</strong>" 的任务或标签</div>
            </div>
            <div v-else>
                <div class="empty-title">暂无任务</div>
                <div class="empty-subtitle">添加第一个待办事项，开始您的高效一天</div>
            </div>
        </div>
        
        <ul class="tdList" v-else>
            <li class="tdItem" v-for="item in todos" :key="item.id">
                <div class="tdItem-main">
                    <input 
                        type="checkbox" 
                        v-model="item.completed" 
                        class="tdToggle" 
                        :id="'todo-' + item.id"
                    />
                    
                    <label :for="'todo-' + item.id" class="checkbox-label"></label>
                    
                    <!-- 优先级颜色标记 -->
                    <span 
                        class="priority-dot" 
                        :class="'priority-' + (item.priority || '中')"
                        @click="$emit('togglePriority', item)"
                        :title="'优先级: ' + (item.priority || '中')"
                    ></span>
                    
                    <span class="tdTxt" :class="{ completed: item.completed }">
                        {{ item.txt }}
                    </span>
                    
                    <!-- 删除按钮 -->
                    <a class="delete-btn" @click.stop="$emit('delTodo', item)" title="删除任务">
                        删除
                    </a>
                    
                    <!-- 标签显示 -->
                    <div class="tags-container" v-if="item.tags && item.tags.length > 0">
                        <span 
                            v-for="tag in item.tags" 
                            :key="tag"
                            class="tag"
                            :style="{ backgroundColor: getTagColor(tag) }"
                            :title="'标签: ' + tag"
                        >
                            {{ tag }}
                        </span>
                    </div>
                </div>
                
                <!-- 标签选择器 -->
                <div class="tag-selector-buttons" v-if="availableTags">
                    <span class="tag-selector-label">选择标签:</span>
                    <button 
                        v-for="tag in availableTags" 
                        :key="tag"
                        class="tag-button"
                        :class="{ active: item.tags && item.tags.includes(tag) }"
                        @click="$emit('toggleTag', item, tag)"
                        :style="{ backgroundColor: getTagColor(tag) }"
                        :title="item.tags && item.tags.includes(tag) ? '移除标签' : '添加标签'"
                    >
                        {{ tag }}
                    </button>
                </div>
            </li>
        </ul>
    </div>
</template>

<script>
export default {
    props: ["todos", "availableTags", "searchMode", "searchKeyword"],
    methods: {
        delTodo(item) {
            this.$emit("delTodo", item);
        },
        getTagColor(tag) {
            const colors = {
                '工作': '#4e6ef2',
                '学习': '#4e6ef2',
                '生活': '#4e6ef2',
                '购物': '#4e6ef2',
                '其他': '#4e6ef2'
            };
            return colors[tag] || '#666';
        }
    }
}
</script>

<style scoped>
.tdContainer {
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  overflow: hidden;
}

.empty-state {
  padding: 40px 20px;
  text-align: center;
  color: #666;
}

.empty-icon {
  font-size: 40px;
  margin-bottom: 10px;
  opacity: 0.5;
}

.empty-title {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: 5px;
}

.empty-subtitle {
  font-size: 14px;
  color: #999;
}

.tdList {
  list-style: none;
  padding: 0;
  text-align: left;
  background-color: #fff;
  border-radius: 10px;
}

.tdItem {
  padding: 10px 20px 10px 10px;
  border-bottom: 1px solid #ddd;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.tdItem:last-child {
  border-bottom: 0;
}

.tdItem-main {
  display: flex;
  align-items: center;
  flex: 1;
}

.tdToggle {
  cursor: pointer;
  margin-right: 10px;
  width: 18px;
  height: 18px;
}

.tdTxt {
  padding-left: 10px;
  flex: 1;
  font-size: 14px;
}

.completed {
  text-decoration: line-through;
  color: #999;
}

.delete-btn {
  color: #ff6b6b;
  cursor: pointer;
  font-size: 12px;
  text-decoration: underline;
  margin-left: 10px;
}

/* 优先级简化 */
.priority-dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  margin-right: 8px;
  cursor: pointer;
}

.priority-高 {
  background-color: #ff4444;
}

.priority-中 {
  background-color: #ffcc00;
}

.priority-低 {
  background-color: #44cc44;
}

/* 标签样式简化 */
.tags-container {
  display: flex;
  gap: 5px;
  margin-left: 10px;
}

.tag {
  padding: 2px 6px;
  border-radius: 3px;
  font-size: 10px;
  color: white;
  background-color: #666;
}

/* 标签选择器简化 */
.tag-selector-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
  margin-top: 5px;
}

.tag-selector-label {
  font-size: 12px;
  color: #666;
  margin-right: 5px;
}

.tag-button {
  padding: 3px 6px;
  border: 1px solid #ddd;
  border-radius: 3px;
  font-size: 10px;
  color: #666;
  cursor: pointer;
  background-color: white;
}

.tag-button.active {
  background-color: #4e6ef2;
  color: white;
  border-color: #4e6ef2;
}
</style>