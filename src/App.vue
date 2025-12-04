<template>
  <div id="app" class="app-container">
    <div class="container">
      <TodoHeader 
        @addTodo="addTodo"
        :availableTags="availableTags"
      ></TodoHeader>
      
      <!-- 搜索框 -->
      <div class="search-box">
        <input 
          type="text" 
          placeholder="输入任务名称或标签，按回车搜索"
          v-model.trim="searchKeyword"
          @keyup.enter="activateSearch"
        />
        <span v-if="searchKeyword || searchActive" class="search-clear" @click="clearSearch">✕</span>
      </div>
      
      <!-- 搜索状态提示 -->
      <div class="search-status" v-if="searchActive && searchKeyword.trim()">
        <span class="search-status-text">
          <span v-if="todoList.length > 0">
            找到 {{ todoList.length }} 个匹配 "{{ searchKeyword }}" 的任务
          </span>
          <span v-else>
            没有找到匹配 "{{ searchKeyword }}" 的任务
          </span>
        </span>
        <button class="search-clear-btn" @click="clearSearch">清除搜索</button>
      </div>
      
      <!-- 标签统计 - 只在非搜索状态显示 -->
      <div class="tag-stats" v-if="Object.keys(tagStats).length > 0 && !searchActive">
        <span class="tag-stat-item" v-for="(count, tag) in tagStats" :key="tag">
          {{ tag }}: {{ count }}
        </span>
      </div>
      
      <!-- 任务列表 -->
      <TodoList 
        :todos="todoList" 
        @delTodo="delTodo"
        @togglePriority="togglePriority"
        @toggleTag="toggleTag"
        :availableTags="availableTags"
        :searchMode="searchActive"
        :searchKeyword="searchKeyword"
      ></TodoList>
      
      <!-- 底部 -->
      <TodoFooter 
        :totalCount="searchActive ? todoList.length : totalCount"
        :activeCount="activeCount"
        :completedCount="completedCount"
        :tabType="tabType"
        @changeTabType="changeTabType"
        @clearCompleted="clearCompleted"
      ></TodoFooter>
    </div>
  </div>
</template>

<script>
import TodoHeader from './components/TodoHeader.vue'
import TodoList from './components/TodoList.vue'
import TodoFooter from './components/TodoFooter.vue'

export default {
  name: 'App',
  components: {
    TodoHeader,
    TodoList,
    TodoFooter
  },
  data() {
    return {
      todos: JSON.parse(localStorage.getItem('vue-todos') || '[]'),
      tabType: 0,
      searchKeyword: '',
      searchActive: false,
      availableTags: ['工作', '学习', '生活', '购物', '其他'],
      priorityOptions: ['低', '中', '高']
    }
  },
  computed: {
    todoList() {
      if (this.searchActive && this.searchKeyword.trim()) {
        return this.getSearchedTodos();
      }
      return this.getTabFilteredTodos();
    },
    
    tagStats() {
      const stats = {};
      this.todos.forEach(todo => {
        if (todo.tags) {
          todo.tags.forEach(tag => {
            stats[tag] = (stats[tag] || 0) + 1;
          });
        }
      });
      return stats;
    },
    
    activeCount() {
      return this.todos.filter(todo => !todo.completed).length;
    },
    
    completedCount() {
      return this.todos.filter(todo => todo.completed).length;
    },
    
    totalCount() {
      return this.todos.length;
    }
  },
  methods: {
    getTabFilteredTodos() {
      let filtered = this.todos;
      
      if (this.tabType === 1) {
        filtered = filtered.filter(item => !item.completed);
      } else if (this.tabType === 2) {
        filtered = filtered.filter(item => item.completed);
      }
      
      return this.sortByPriority(filtered);
    },
    
    getSearchedTodos() {
      const keyword = this.searchKeyword.toLowerCase().trim();
      
      if (!keyword) return [];
      
      let filtered = this.todos.filter(item => {
        if (item.txt && item.txt.toLowerCase().includes(keyword)) {
          return true;
        }
        
        if (item.tags && item.tags.length > 0) {
          for (let i = 0; i < item.tags.length; i++) {
            if (item.tags[i].toLowerCase().includes(keyword)) {
              return true;
            }
          }
        }
        
        return false;
      });
      
      return this.sortByPriority(filtered);
    },
    
    sortByPriority(todos) {
      const priorityOrder = { '高': 3, '中': 2, '低': 1, '无': 0 };
      return [...todos].sort((a, b) => {
        const aPriority = priorityOrder[a.priority || '无'];
        const bPriority = priorityOrder[b.priority || '无'];
        return bPriority - aPriority;
      });
    },
    
    addTodo(newTodo) {
      const task = {
        id: Date.now(),
        txt: newTodo.text || newTodo,
        completed: false,
        priority: '中',
        tags: newTodo.tags || [],
        createdAt: new Date().toISOString()
      };
      
      this.todos.push(task);
    },
    
    delTodo(delItem) {
      const index = this.todos.findIndex(item => item.id === delItem.id);
      if (index !== -1) {
        this.todos.splice(index, 1);
      }
    },
    
    changeTabType(type) {
      this.tabType = type;
    },
    
    togglePriority(item) {
      const currentPriority = item.priority || '中';
      const currentIndex = this.priorityOptions.indexOf(currentPriority);
      const nextIndex = (currentIndex + 1) % this.priorityOptions.length;
      item.priority = this.priorityOptions[nextIndex];
    },
    
    toggleTag(item, tag) {
      if (!item.tags) {
        item.tags = [];
      }
      const tagIndex = item.tags.indexOf(tag);
      if (tagIndex > -1) {
        item.tags.splice(tagIndex, 1);
      } else {
        item.tags.push(tag);
      }
    },
    
    activateSearch() {
      const keyword = this.searchKeyword.trim();
      if (keyword) {
        this.searchActive = true;
      } else {
        this.clearSearch();
      }
    },
    
    clearSearch() {
      this.searchKeyword = '';
      this.searchActive = false;
    },
    
    clearCompleted() {
      this.todos = this.todos.filter(todo => !todo.completed);
    }
  },
  watch: {
    todos: {
      handler(todos) {
        if (this.saveTimeout) {
          clearTimeout(this.saveTimeout);
        }
        this.saveTimeout = setTimeout(() => {
          localStorage.setItem('vue-todos', JSON.stringify(todos));
        }, 500);
      },
      deep: true
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body {
  background-color: #f5f5f5;
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}

.container {
  max-width: 600px;
  min-height: 100%;
  margin: 0 auto;
  padding: 20px;
}

/* 搜索框样式简化 */
.search-box {
  margin: 15px 0;
}

.search-box input {
  width: 100%;
  padding: 12px 15px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
  background-color: white;
}

.search-clear {
  position: absolute;
  right: 15px;
  top: 50%;
  transform: translateY(-50%);
  cursor: pointer;
  color: #999;
  font-size: 16px;
}

/* 搜索状态提示简化 */
.search-status {
  margin: 10px 0;
  padding: 10px;
  background-color: #f8f9fa;
  border-radius: 6px;
  font-size: 14px;
}

.search-clear-btn {
  padding: 5px 10px;
  background-color: #ff6b6b;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
}

/* 标签统计简化 */
.tag-stats {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin: 10px 0;
}

.tag-stat-item {
  padding: 4px 8px;
  background-color: #e9ecef;
  border-radius: 4px;
  font-size: 12px;
  color: #495057;
}
</style>