<template>
  <v-app>
    <v-app-bar color="purple" flat>
      <v-app-bar-title class="text-center">To Do</v-app-bar-title>
    </v-app-bar>
    <v-main>
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
    </v-main>
  </v-app>
</template>

<script>
import uuid4 from "uuid4";

const TODO_STORAGE = 'todoStorage'
const SetTodoStorage = (todos) => localStorage.setItem(TODO_STORAGE, JSON.stringify(todos))
const todayDate = () => new Date().toLocaleDateString('de-CH')
const todoId = () => uuid4()
const filters = {
  all: (todos) => todos,
  active: (todos) => todos.filter((todo) => !todo.isDone),
  done: (todos) => todos.filter((todo) => todo.isDone)
}
export default {
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

      this.todos.unshift(newTodo)
      SetTodoStorage(this.todos)
      this.todoInput = ''
    },
    checkTodo(todo){
      console.log("Todo Checked")
      const todoIndex = this.todos.indexOf(todo)
      this.todos[todoIndex].isDone = !this.todos[todoIndex].isDone

      SetTodoStorage(this.todos)
    },
    deleteTodo(todo){
      console.log("Todo Deleted")
      const todoIndex = this.todos.indexOf(todo)
      this.todos.splice(todoIndex, 1)
      SetTodoStorage(this.todos)
    }
  }
}
</script>
