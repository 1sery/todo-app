<template>
    <div class="hdContainer">
        <h1 class="hdTitle">待办事项</h1>
        
        <!-- 标签选择器 -->
        <div class="tag-selector-container" v-if="availableTags && availableTags.length > 0">
            <div class="tag-buttons">
                <button 
                    v-for="tag in availableTags" 
                    :key="tag"
                    class="tag-button"
                    :class="{ active: selectedTags.includes(tag) }"
                    @click="toggleTag(tag)"
                    :style="{ backgroundColor: getTagColor(tag) }"
                >
                    {{ tag }}
                </button>
            </div>
            <div class="selected-tags" v-if="selectedTags.length > 0">
                已选标签: {{ selectedTags.join(', ') }}
                <span class="clear-tags" @click="clearTags">清除</span>
            </div>
        </div>
        
        <div class="input-container">
            <input 
                class="newTodo" 
                placeholder="请输入待办事项，按回车添加" 
                v-model.trim="newTodo" 
                @keyup.enter="addNewTodo" 
                ref="todoInput"
            />
            <button class="add-btn" @click="addNewTodo" title="添加任务"></button>
        </div>
        <p v-show="isShowMsg" class="hdMsg">
            最多输入{{ countLimit }}个字符!!!
        </p>
    </div>
</template>

<script>
const WORD_COUNT_LIMIT = 50;
export default {
    props: ["availableTags"],
    data() {
        return {
            newTodo: "",
            countLimit: WORD_COUNT_LIMIT,
            selectedTags: []
        }
    },
    computed: {
        isShowMsg() {
            return this.newTodo.length > WORD_COUNT_LIMIT;
        }
    },
    methods: {
        addNewTodo() {
            if (this.newTodo.length == 0 || this.newTodo.length > WORD_COUNT_LIMIT) {
                return;
            }
            
            const task = {
                text: this.newTodo,
                tags: [...this.selectedTags]
            };
            
            this.$emit("addTodo", task);
            this.newTodo = "";
            this.selectedTags = [];
            this.$nextTick(() => {
                this.$refs.todoInput.focus();
            });
        },
        toggleTag(tag) {
            const index = this.selectedTags.indexOf(tag);
            if (index > -1) {
                this.selectedTags.splice(index, 1);
            } else {
                this.selectedTags.push(tag);
            }
        },
        clearTags() {
            this.selectedTags = [];
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
.hdContainer {
  text-align: center;
  font-size: 16px;
  margin-bottom: 20px;
}

.hdTitle {
  color: #4e6ef2;
  margin-bottom: 15px;
  font-size: 24px;
}

.input-container {
  position: relative;
  margin-bottom: 10px;
}

.newTodo {
  width: 100%;
  padding: 20px 20px;
  border: none;
  border-radius: 10px;
  font-size: 16px;
  box-sizing: border-box;
  background-color: white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.newTodo:focus {
  outline-color: #4e6ef2;
  box-shadow: 0 2px 8px rgba(78, 110, 242, 0.2);
}

.add-btn {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  width: 36px;
  height: 36px;
  border: none;
  background-color: #4e6ef2;
  color: white;
  border-radius: 50%;
  font-size: 20px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.hdMsg {
  color: red;
  margin: 10px 0;
  font-size: 14px;
}

/* 标签选择器简化 */
.tag-selector-container {
  margin-bottom: 15px;
}

.tag-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
  justify-content: center;
  margin-bottom: 10px;
}

.tag-button {
  padding: 5px 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 12px;
  color: #666;
  cursor: pointer;
  background-color: white;
}

.tag-button.active {
  background-color: #4e6ef2;
  color: white;
  border-color: #4e6ef2;
}

.selected-tags {
  font-size: 12px;
  color: #666;
}

.clear-tags {
  margin-left: 10px;
  color: #ff6b6b;
  cursor: pointer;
  text-decoration: underline;
}
</style>