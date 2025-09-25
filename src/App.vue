<template>
  <div id="app">
    <h1>Todo App</h1>
    <div class="todo-input">
      <input 
        v-model="newTodo" 
        @keyup.enter="addTodo"
        placeholder="Add a new todo..."
        type="text"
      />
      <button @click="addTodo">Add</button>
    </div>
    
    <div class="todos">
      <div 
        v-for="todo in todos" 
        :key="todo.id" 
        class="todo-item"
        :class="{ completed: todo.completed }"
      >
        <span @click="toggleTodo(todo.id)">{{ todo.text }}</span>
        <button @click="deleteTodo(todo.id)" class="delete-btn">Delete</button>
      </div>
    </div>
    
    <div class="status">
      API Status: <span :class="{ online: apiOnline, offline: !apiOnline }">
        {{ apiOnline ? 'Online' : 'Offline' }}
      </span>
    </div>
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
      apiOnline: false
    }
  },
  async mounted() {
    await this.checkApiStatus()
    await this.fetchTodos()
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
        const formData = new FormData()
        formData.append('todo_text', this.newTodo)
        
        const response = await axios.post(`${API_BASE}/todos`, formData)
        this.todos.push(response.data)
        this.newTodo = ''
      } catch (error) {
        console.error('Error adding todo:', error)
      }
    },
    async toggleTodo(id) {
      try {
        const response = await axios.put(`${API_BASE}/todos/${id}`)
        const index = this.todos.findIndex(todo => todo.id === id)
        if (index !== -1) {
          this.todos[index] = response.data
        }
      } catch (error) {
        console.error('Error toggling todo:', error)
      }
    },
    async deleteTodo(id) {
      try {
        await axios.delete(`${API_BASE}/todos/${id}`)
        this.todos = this.todos.filter(todo => todo.id !== id)
      } catch (error) {
        console.error('Error deleting todo:', error)
      }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  max-width: 600px;
  margin: 50px auto;
  padding: 20px;
}

h1 {
  text-align: center;
  color: #2c3e50;
}

.todo-input {
  display: flex;
  margin-bottom: 20px;
}

.todo-input input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px 0 0 4px;
}

.todo-input button {
  padding: 10px 15px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 0 4px 4px 0;
  cursor: pointer;
}

.todo-input button:hover {
  background: #369870;
}

.todo-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  border: 1px solid #eee;
  margin-bottom: 5px;
  border-radius: 4px;
}

.todo-item span {
  cursor: pointer;
  flex: 1;
}

.todo-item.completed span {
  text-decoration: line-through;
  opacity: 0.6;
}

.delete-btn {
  background: #e74c3c;
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
}

.delete-btn:hover {
  background: #c0392b;
}

.status {
  margin-top: 20px;
  text-align: center;
  font-weight: bold;
}

.online {
  color: #27ae60;
}

.offline {
  color: #e74c3c;
}
</style>