<template >
  <h1 @keyup.meta.z="redo">{{ today }}</h1>
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
      :key="todo.id"
      v-motion
    >
      <div>
        <input type="checkbox" v-model="todo.done" />
        {{ todo.title }}
      </div>
      <div class="estimation" @click="() => openEstimationPopUp(todo.id)">{{ todo.estimation }}min
        <div v-if="editTodoEstimationId === todo.id" class="estimation-popup" @click.stop >
          <input type="number" placeholder="Enter a new estimation" />
        </div>
      </div>
    </div>
    </TransitionGroup>
  </section>
</template>

<script>
import dayjs from "dayjs";
import { nanoid } from "nanoid";

export default {
  name: "App",
  data() {
    return {
      newTodo: "",
      today: dayjs().format("dddd MMM, YYYY"),
      todoHistory: [],
      todos: JSON.parse(window.localStorage.getItem("todos")) || [],
      editTodoEstimationId: null,
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
        { done: false, title: splitTitle[0] || "New Task", estimation: time, id: nanoid() },
      ];
      this.newTodo = "";
    },
    redo() {
      console.log("redooo")
    },
    openEstimationPopUp(todoId) {
      if (this.editTodoEstimationId !== null) {
        this.editTodoEstimationId = null;
      } else {
        this.editTodoEstimationId = todoId;
      }
    },
    editEstimation(todoId, newEstimation) {
      this.todos = [...this.todos.map(todo => {
        if (todo.id === todoId) {
          return {...todo, estimation: newEstimation}
        } else {
          return todo;
        }
      })]
    }
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

.list-move, 
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}

.list-move {
  transition-delay: 250ms;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
}

/* ensure leaving items are taken out of layout flow so that moving
   animations can be calculated correctly. */
.list-leave-active {
  position: absolute;
}

.estimation {
  cursor: pointer;
  position: relative;
}

.estimation-popup {
  /*position: absolute;
  top: 25px;
  z-index: 5;
*/}
</style>
