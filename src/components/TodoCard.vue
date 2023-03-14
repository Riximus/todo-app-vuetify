<template>
  <v-card class="mx-auto" max-width="400" variant="flat">
  <v-form @submit.prevent="addTodo">
    <v-text-field label="I want to ..."
                  placeholder="water my plants"
                  autofocus
                  variant="underlined"
                  append-inner-icon="mdi-send"
                  class="mt-2"
                  hide-details
                  v-model.trim="todoInput"
                  @click:append-inner="addTodo"
    />
  </v-form>

  <v-list>
    <div class="d-flex justify-space-between">
      <v-list-subheader>Your tasks: {{todosDone}} / {{todos.length}}</v-list-subheader>
      <v-card-actions>
        <v-btn-toggle mandatory v-model="filterState">
          <v-btn value="all">all</v-btn>
          <v-btn value="done">done</v-btn>
          <v-btn value="active">active</v-btn>
        </v-btn-toggle>
      </v-card-actions>
    </div>

    <v-progress-linear :model-value="todosDone" :max="todos.length"></v-progress-linear>

    <v-list-item v-for="todo in todosFiltered" :key="todos.id" class="my-2" @click="checkTodo(todo)">
      <div class="d-flex justify-space-between">
        <v-list-item-action>
          <v-checkbox-btn @click.stop="checkTodo(todo)" v-model="todo.isDone"/>
        </v-list-item-action>
        <div :class="{'text-decoration-line-through text-disabled': todo.isDone}">
          <v-list-item-title class="text-center">{{todo.task}}</v-list-item-title>
          <v-list-item-subtitle class="text-center">{{todo.date}}</v-list-item-subtitle>
        </div>
        <v-btn @click.stop="deleteTodo(todo)" variant="plain" height="auto">
          <v-icon size="x-large" color="red" icon="mdi-delete"></v-icon>
        </v-btn>
      </div>
    </v-list-item>

  </v-list>

  </v-card>
</template>

<script>
import uuid4 from "uuid4";

const TODO_STORAGE = 'todoStorage'
const SetTodoStorage = (todos) => localStorage.setItem(TODO_STORAGE, JSON.stringify(todos))
const todayDate = () => new Date().toLocaleDateString('en-CA') // yyyy-mm-dd

const todoId = () => uuid4()
const SERVER_URL_TODOS = import.meta.env.VITE_URL_SERVER + '/todos'
const filters = {
  all: (todos) => todos,
  active: (todos) => todos.filter((todo) => !todo.isDone),
  done: (todos) => todos.filter((todo) => todo.isDone)
}
export default {
  name: "TodoCard",
  data(){
    return{
      todoInput: '',
      todos: JSON.parse(localStorage.getItem(TODO_STORAGE)) || [],
      filterState: 'all'
    }
  },
  computed:{
    todosFiltered(){
      return filters[this.filterState](this.todos)
    },
    todosDone(){
      return filters.done(this.todos).length
    }
  },
  methods:{
    addTodo(){
      console.log("Todo Added")

      if (!this.todoInput) return

      let newTodo = {
        id: todoId(),
        task: this.todoInput,
        date: todayDate(),
        isDone: false
      }

      fetch(SERVER_URL_TODOS, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(newTodo)
      })

      this.todos.unshift(newTodo)
      SetTodoStorage(this.todos)
      this.todoInput = ''
    },
    checkTodo(todo){
      console.log("Todo Checked")
      let todoIndex = this.todos.indexOf(todo)
      this.todos[todoIndex].isDone = !this.todos[todoIndex].isDone

      const id = this.todos[todoIndex].id
      let completedTodo = {
        isDone: this.todos[todoIndex].isDone
      }

      console.log(id)
      fetch(`${SERVER_URL_TODOS}/${id}`, {
        method: 'PATCH',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(completedTodo)
      })
      SetTodoStorage(this.todos)
    },
    deleteTodo(todo){
      console.log("Todo Deleted")
      const todoIndex = this.todos.indexOf(todo)
      let id = this.todos[todoIndex].id
      this.todos.splice(todoIndex, 1)

      fetch(`${SERVER_URL_TODOS}/${id}`, {
        method: 'DELETE'
      })
      SetTodoStorage(this.todos)
    }
  }
}
</script>

<style scoped>

</style>
