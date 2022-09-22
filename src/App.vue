<template>
  <h1>{{ today }}</h1>
  <div class="details">
    <small
      ><strong>{{ todos.filter((todo) => !todo.done).length }}</strong>
      task<span v-if="this.todos.filter((todo) => !todo.done).length > 1"
        >s</span
      ></small
    ><small
      >End at <strong>{{ endTime }}</strong></small
    >
  </div>
  <input
    class="new-todo"
    placeholder="Enter a new todo with time estimate"
    v-model="this.newTodo"
    @keyup.enter="this.new"
  />
  <section class="todos">
    <TransitionGroup name="list" tag="todooos">
    <div
      class="todo"
      v-for="todo in this.todos.filter((todo) => !todo.done)"
      :key="todo.title"
      v-motion
      :initial="{
        opacity: 0,
      }"
      :enter="{
        opacity: 1,
      }"
    >
      <div>
        <input type="checkbox" v-model="todo.done" />
        {{ todo.title }}
      </div>
      <div>{{ todo.estimation }}min</div>
    </div>
    </TransitionGroup>
  </section>
</template>

<script>
import dayjs from "dayjs";

export default {
  name: "App",
  data() {
    return {
      newTodo: "",
      today: dayjs().format("dddd MMM, YYYY"),
      todos: JSON.parse(window.localStorage.getItem("todos")) || [],
    };
  },
  computed: {
    endTime() {
      return dayjs()
        .add(
          this.todos
            .filter((todo) => !todo.done)
            .reduce((previous, current) => previous + current.estimation, 0),
          "minutes"
        )
        .format("h:mmA");
    },
  },
  watch: {
    todos: {
      handler() {
        window.localStorage.setItem("todos", JSON.stringify(this.todos));
      },
      deep: true,
    },
  },
  methods: {
    new() {
      const splitTitle = this.newTodo.split(" for ");
      let time = 0;
      if (!isNaN(parseInt(splitTitle[1]))) {
        time = parseInt(splitTitle[1]);
      }
      this.todos = [
        ...this.todos,
        { done: false, title: splitTitle[0] || "New Task", estimation: time },
      ];
      this.newTodo = "";
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  max-width: 400px;
  margin: 0 auto;
}

.details {
  display: flex;
  justify-content: space-between;
}

.new-todo {
  margin-top: 15px;
  width: 100%;
}

.todos {
  margin-top: 15px;
}

.todo {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.list-move, /* apply transition to moving elements */
.list-leave-active {
  transition: all 0.3s ease;
}


.list-enter-active {
  transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

/* ensure leaving items are taken out of layout flow so that moving
   animations can be calculated correctly. */
.list-leave-active {
  position: absolute;
}
</style>
