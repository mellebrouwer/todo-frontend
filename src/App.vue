<template>
  <div id="app">
    <header>
      <h1>✨ My Todo App</h1>
      <div class="stats" v-if="stats">
        <span class="stat-item">📝 {{ stats.total }} total</span>
        <span class="stat-item">✅ {{ stats.completed }} done</span>
        <span class="stat-item">📊 {{ stats.completion_rate }}% complete</span>
      </div>
    </header>
    
    <div class="todo-input">
      <input 
        v-model="newTodo" 
        @keyup.enter="addTodo"
        placeholder="What needs to be done? 🚀"
        type="text"
        :disabled="!apiOnline"
      />
      <button @click="addTodo" :disabled="!apiOnline || !newTodo.trim()">Add Todo</button>
    </div>
    
    <div class="filter-buttons">
      <button 
        v-for="filter in filters" 
        :key="filter.key"
        @click="currentFilter = filter.key"
        :class="{ active: currentFilter === filter.key }"
        class="filter-btn"
      >
        {{ filter.label }}
      </button>
    </div>
    
    <div class="todos" v-if="filteredTodos.length">
      <div 
        v-for="todo in filteredTodos" 
        :key="todo.id" 
        class="todo-item"
        :class="{ completed: todo.completed }"
      >
        <div class="todo-content">
          <span @click="toggleTodo(todo.id)" class="todo-text">
            {{ todo.completed ? '✅' : '⭕' }} {{ todo.text }}
          </span>
          <div class="todo-meta">
            <small>Created: {{ formatDate(todo.created_at) }}</small>
            <small v-if="todo.completed && todo.completed_at">
              Completed: {{ formatDate(todo.completed_at) }}
            </small>
          </div>
        </div>
        <button @click="deleteTodo(todo.id)" class="delete-btn">🗑️</button>
      </div>
    </div>
    
    <div v-else-if="todos.length" class="empty-state">
      <p>No todos match the current filter 🔍</p>
    </div>
    
    <div v-else class="empty-state">
      <p>No todos yet! Add one above to get started 🎯</p>
    </div>
    
    <footer class="status">
      <div class="api-status">
        API Status: <span :class="{ online: apiOnline, offline: !apiOnline }">
          {{ apiOnline ? '🟢 Online' : '🔴 Offline' }}
        </span>
      </div>
      <button @click="refreshData" class="refresh-btn" :disabled="!apiOnline">🔄 Refresh</button>
    </footer>
  </div>
</template>

<script>
import axios from 'axios'

const API_BASE = import.meta.env.VITE_API_URL || 'http://localhost:8000'

export default {
  name: 'App',
  data() {
    return {
      todos: [],
      newTodo: '',
      apiOnline: false,
      stats: null,
      currentFilter: 'all',
      filters: [
        { key: 'all', label: 'All' },
        { key: 'active', label: 'Active' },
        { key: 'completed', label: 'Completed' }
      ]
    }
  },
  computed: {
    filteredTodos() {
      switch (this.currentFilter) {
        case 'active':
          return this.todos.filter(todo => !todo.completed)
        case 'completed':
          return this.todos.filter(todo => todo.completed)
        default:
          return this.todos
      }
    }
  },
  async mounted() {
    await this.refreshData()
  },
  methods: {
    async checkApiStatus() {
      try {
        await axios.get(`${API_BASE}/`)
        this.apiOnline = true
      } catch (error) {
        this.apiOnline = false
        console.error('API is offline:', error)
      }
    },
    async fetchTodos() {
      try {
        const response = await axios.get(`${API_BASE}/todos`)
        this.todos = response.data
      } catch (error) {
        console.error('Error fetching todos:', error)
      }
    },
    async addTodo() {
      if (!this.newTodo.trim()) return
      
      try {
        const response = await axios.post(`${API_BASE}/todos`, {
          text: this.newTodo.trim()
        })
        await this.fetchTodos()
        await this.fetchStats()
        this.newTodo = ''
      } catch (error) {
        console.error('Error adding todo:', error)
      }
    },
    async toggleTodo(id) {
      try {
        await axios.put(`${API_BASE}/todos/${id}`)
        await this.fetchTodos()
        await this.fetchStats()
      } catch (error) {
        console.error('Error toggling todo:', error)
      }
    },
    async deleteTodo(id) {
      if (!confirm('Are you sure you want to delete this todo?')) return
      
      try {
        await axios.delete(`${API_BASE}/todos/${id}`)
        await this.fetchTodos()
        await this.fetchStats()
      } catch (error) {
        console.error('Error deleting todo:', error)
      }
    },
    async fetchStats() {
      try {
        const response = await axios.get(`${API_BASE}/stats`)
        this.stats = response.data
      } catch (error) {
        console.error('Error fetching stats:', error)
      }
    },
    async refreshData() {
      await this.checkApiStatus()
      if (this.apiOnline) {
        await this.fetchTodos()
        await this.fetchStats()
      }
    },
    formatDate(dateString) {
      return new Date(dateString).toLocaleString()
    }
  }
}
</script>

<style>
#app {
  font-family: 'SF Pro Display', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  max-width: 800px;
  margin: 20px auto;
  padding: 20px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  color: #333;
}

header {
  text-align: center;
  margin-bottom: 30px;
  background: rgba(255, 255, 255, 0.95);
  padding: 20px;
  border-radius: 15px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
}

h1 {
  color: #2c3e50;
  margin: 0 0 15px 0;
  font-size: 2.5em;
  font-weight: 700;
}

.stats {
  display: flex;
  justify-content: center;
  gap: 20px;
  flex-wrap: wrap;
}

.stat-item {
  background: #f8f9fa;
  padding: 8px 16px;
  border-radius: 20px;
  font-size: 0.9em;
  font-weight: 600;
  border: 2px solid #e9ecef;
}

.todo-input {
  display: flex;
  margin-bottom: 30px;
  background: rgba(255, 255, 255, 0.95);
  padding: 20px;
  border-radius: 15px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
}

.todo-input input {
  flex: 1;
  padding: 15px;
  border: 2px solid #e9ecef;
  border-radius: 10px 0 0 10px;
  font-size: 16px;
  outline: none;
  transition: border-color 0.3s;
}

.todo-input input:focus {
  border-color: #667eea;
}

.todo-input button {
  padding: 15px 25px;
  background: linear-gradient(45deg, #667eea, #764ba2);
  color: white;
  border: none;
  border-radius: 0 10px 10px 0;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s;
}

.todo-input button:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
}

.todo-input button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.filter-buttons {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
}

.filter-btn {
  padding: 10px 20px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  background: rgba(255, 255, 255, 0.1);
  color: white;
  border-radius: 25px;
  cursor: pointer;
  transition: all 0.3s;
  font-weight: 600;
}

.filter-btn:hover, .filter-btn.active {
  background: rgba(255, 255, 255, 0.9);
  color: #667eea;
  border-color: white;
}

.todos {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 15px;
  padding: 20px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

.todo-item {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 20px;
  border: 1px solid #f1f3f4;
  margin-bottom: 10px;
  border-radius: 12px;
  background: #fafbfc;
  transition: all 0.3s;
}

.todo-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.1);
}

.todo-content {
  flex: 1;
  margin-right: 15px;
}

.todo-text {
  cursor: pointer;
  font-size: 16px;
  font-weight: 500;
  line-height: 1.4;
  display: block;
  margin-bottom: 8px;
  transition: all 0.3s;
}

.todo-item.completed .todo-text {
  opacity: 0.6;
  text-decoration: line-through;
}

.todo-meta {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.todo-meta small {
  color: #6c757d;
  font-size: 12px;
}

.delete-btn {
  background: #ff6b6b;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 16px;
  transition: all 0.3s;
  flex-shrink: 0;
}

.delete-btn:hover {
  background: #ff5252;
  transform: scale(1.1);
}

.empty-state {
  text-align: center;
  padding: 40px;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 15px;
  margin-bottom: 20px;
}

.empty-state p {
  font-size: 18px;
  color: #6c757d;
  margin: 0;
}

footer {
  background: rgba(255, 255, 255, 0.95);
  padding: 20px;
  border-radius: 15px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 15px;
}

.api-status {
  font-weight: bold;
  font-size: 16px;
}

.refresh-btn {
  background: linear-gradient(45deg, #667eea, #764ba2);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 25px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s;
}

.refresh-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
}

.refresh-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.online {
  color: #28a745;
}

.offline {
  color: #dc3545;
}

@media (max-width: 768px) {
  #app {
    margin: 10px;
    padding: 15px;
  }
  
  .stats {
    flex-direction: column;
    align-items: center;
  }
  
  .todo-input {
    flex-direction: column;
  }
  
  .todo-input input,
  .todo-input button {
    border-radius: 10px;
    margin-bottom: 10px;
  }
  
  footer {
    flex-direction: column;
    text-align: center;
  }
}
</style>