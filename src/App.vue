<script setup>
  import { ref, onMounted, computed, watch } from 'vue'

  const todos = ref([]);
  const name = ref('');

  const input_content = ref('');
  const input_category = ref(null);
  const filter_category = ref(null);

  const todos_filtered = computed(() => todos.value.sort((a, b) => b.createdAt - a.createdAt)
                                                   .filter(todo => {
                                                      return !filter_category.value || filter_category.value == 'all' ?
                                                      todo.category == "work" || todo.category == "personal" :
                                                      todo.category == filter_category.value
                                                    }));
                                                  

  const setFilterCategory = (e) => {
    filter_category.value = e.target.value;
    console.log(filter_category)
    console.log(todos_filtered);
  }

  const addTodo = () => {
    if (input_content.value.trim() === '' || input_category.value === null) {
      return;
    }

    todos.value.push({
      content: input_content.value,
      category: input_category.value,
      done: false,
      createdAt: new Date().getTime()
    })

    input_content.value = '';
    input_category.value = null;
  };

  const removeTodo = todo => {
    todos.value = todos.value.filter(t => t !== todo);
  }

  // save to localStorage everytime name or todos change
  watch(todos, newVal => {
    localStorage.setItem('todos', JSON.stringify(newVal));
  }, { deep: true });

  watch(name, (newVal) => {
    localStorage.setItem('name', newVal);
  });

  onMounted(() => {
    name.value = localStorage.getItem('name') || '';
    todos.value = JSON.parse(localStorage.getItem('todos')) || [];
  })
</script>

<template>
  <main class="app">
    <section class="greeting">
      <h2 class="title">
        What's up, <input type="text" placeholder="Name here" v-model="name" />
      </h2>
    </section>

    <section class="create-todo">
      <h3>CREATE A TODO</h3>

      <form @submit.prevent="addTodo">
        <h4>What's on your todo list?</h4>
        <input 
          type="text" 
          placeholder="e.g. make a video"
          v-model="input_content"
        />
        
        <h4>Pick a category</h4>

        <div class="options">

          <label>
            <input 
              type="radio" 
              name="category"
              value="work"
              v-model="input_category" 
            />
            <span class="bubble work"></span>
            <div>Work</div>
          </label>

          <label>
            <input 
              type="radio" 
              name="category"
              value="personal"
              v-model="input_category" 
            />
            <span class="bubble personal"></span>
            <div>Personal</div>
          </label>

        </div>

        <input type="submit" value="Add todo" />
      </form>
    </section>

    <section class="todo-list">
      <h3>TODO LIST</h3>
      <div class="filter">
        <p>Filter: </p>
        <button 
          class="filter-all-btn"
          :class="{ 'all-selected': filter_category == 'all'}"
          value="all"
          @click="setFilterCategory"
        >
          Select All
        </button>
        <button 
          class="filter-work-btn"
          :class="{ 'work-selected': filter_category == 'work'}"
          value="work" 
          @click="setFilterCategory"
        >
          Work
        </button>
        <button 
          class="filter-personal-btn"
          :class="{ 'personal-selected': filter_category == 'personal'}"
          value="personal" 
          @click="setFilterCategory"
        >
          Personal
        </button>
      </div>
      <div class="list">
        <div 
          v-for="todo in todos_filtered" 
          :class="`todo-item ${todo.done && 'done'}`"
        >
          <label>
            <input type="checkbox" v-model="todo.done" />
            <span :class="`bubble ${todo.category}`"></span>
          </label>

          <div class="todo-content">
            <input type="text" v-model="todo.content" />
          </div>

          <div class="actions">
            <button class="delete" @click="removeTodo(todo)">Delete</button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>