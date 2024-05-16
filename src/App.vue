<script setup>
import { ref, onMounted, computed, watch } from 'vue'
import axios from 'axios'

const todos = ref([])
const name = ref('')

const input_content = ref('')
const input_category = ref(null)

const todos_asc = computed(() => todos.value.sort((a,b) =>{
    return a.createdAt - b.createdAt
}))

const activeTab = ref('todos') // Default active tab is 'todos'

watch(name, (newVal) => {
    localStorage.setItem('name', newVal)
})

watch(todos, (newVal) => {
    localStorage.setItem('todos', JSON.stringify(newVal))
}, {
    deep: true
})

const addTodo = () => {
    if (input_content.value.trim() === '' || input_category.value === null) {
        return
    }

    todos.value.push({
        content: input_content.value,
        category: input_category.value,
        done: false,
        editable: false,
        createdAt: new Date().getTime()
    })
}

const removeTodo = (todo) => {
    todos.value = todos.value.filter((t) => t !== todo)
}

onMounted(() => {
    name.value = localStorage.getItem('name') || ''
    todos.value = JSON.parse(localStorage.getItem('todos')) || []
})

const users = ref([])
const selectedUser = ref(null)
const posts = ref([])

// Fetch users from JSONPlaceholder API
onMounted(async () => {
    try {
        const response = await axios.get('https://jsonplaceholder.typicode.com/users')
        users.value = response.data
    } catch (error) {
        console.error('Error fetching users:', error)
    }
})

// Fetch posts when user is selected
const fetchPosts = async () => {
    if (!selectedUser.value) return
    try {
        const response = await axios.get(`https://jsonplaceholder.typicode.com/posts?userId=${selectedUser.value}`)
        posts.value = response.data
    } catch (error) {
        console.error('Error fetching posts:', error)
    }
}
</script>

<template>
    <main class="app">
        
        <header class="header">
            <nav>
                <ul>
                    <li @click="activeTab = 'todos'" :class="{ 'active': activeTab === 'todos' }">Todos</li>
                    <li @click="activeTab = 'post'" :class="{ 'active': activeTab === 'post' }">Post</li>
                </ul>
            </nav>
        </header>

        <section v-if="activeTab === 'todos'" class="todos-section">
            <section class="greeting">
                <h2 class="title">
                    Assalamualaikum, <input type="text" id="name" placeholder="Tulis Nama" v-model="name">
                </h2>
            </section>
			<section class="create-todo">
			<h3> KEGIATAN HARI INI </h3>

			<form id="new-todo-form" @submit.prevent="addTodo">
				<h4>Apa yang akan kamu lakukan hari ini?</h4>
				<input 
					type="text" 
					name="content" 
					id="content" 
					placeholder="Tulis kegiatan kamu hari ini"
					v-model="input_content" />
				
				<h4>Pilih warna agar menarik</h4>
				<div class="options">

					<label>
						<input 
							type="radio" 
							name="category" 
							id="category1" 
							value="business"
							v-model="input_category" />
						<span class="bubble business"></span>
						<div></div>
					</label>

					<label>
						<input 
							type="radio" 
							name="category" 
							id="category2" 
							value="personal"
							v-model="input_category" />
						<span class="bubble personal"></span>
						<div></div>
					</label>

				</div>

				<input type="submit" value="Tambahkan" />
			</form>
		</section>

		<section class="todo-list">
			  <h3> Kegiatan kamu </h3>

			<div class="list" id="todo-list">

				<div v-for="todo in todos_asc" :class="`todo-item ${todo.done && 'done'}`">
					<label>
						<input type="checkbox" v-model="todo.done" />
						<span :class="`bubble ${
							todo.category == 'business' 
								? 'business' 
								: 'personal'
						}`"></span>
					</label>

					<div class="todo-content">
						<input type="text" v-model="todo.content" />
					</div>

					<div class="actions">
						<button class="delete" @click="removeTodo(todo)">Hapus</button>
					</div>
				</div>

			</div>
		</section>

            <section class="create-todo">
                <!-- Todo creation form -->
            </section>

            <section class="todo-list">
                <!-- Todo list -->
            </section>
        </section>

        <section v-else-if="activeTab === 'post'" class="post-section">
            <h3>Select User</h3>
				
            <select v-model="selectedUser" @change="fetchPosts">
                <option disabled value="">Please select a user</option>
                <option v-for="user in users" :key="user.id" :value="user.id">{{ user.name }}</option>
            </select>

            <div v-if="selectedUser">
                <h3>Posts by {{ users.find(u => u.id === selectedUser)?.name }}</h3>
                <ul>
                    <li v-for="post in posts" :key="post.id">
                        <h4>{{ post.title }}</h4>
                        <p>{{ post.body }}</p>
                    </li>
                </ul>
            </div>
        </section>
    </main>


 
		
</template>