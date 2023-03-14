<template>
  <v-app>
    <v-app-bar color="purple" flat>
      <v-app-bar-title class="text-center">To Do</v-app-bar-title>
    </v-app-bar>
    <v-main>
      <TodoCard/>
    </v-main>
  </v-app>
</template>

<script>
import TodoCard from "@/components/TodoCard.vue";

export default {
  components: {TodoCard}


}
</script>
