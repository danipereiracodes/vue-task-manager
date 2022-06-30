<template>
  <Tasks
    @toggle-reminder="toggleReminder"
    @delete-task="deleteTask"
    :tasks="tasks"
  />
  <div><AddTask @add-task="addTask" v-show="showAddTask" /></div>
</template>

<script>
import Tasks from "../components/Tasks";
import AddTask from "../components/AddTask";
export default {
  name: "Home-main",
  components: {
    Tasks,
    AddTask,
  },
  props: {
    showAddTask: Boolean,
  },
  data() {
    return { tasks: [] };
  },
  methods: {
    async fetchTasks() {
      const res = await fetch("api/tasks");
      const data = await res.json();
      console.log(data);
      return data;
    },

    async fetchTask(id) {
      const res = await fetch(`api/tasks/${id}`);
      const data = await res.json();

      return data;
    },

    async addTask(task) {
      const res = await fetch("api/tasks", {
        method: "POST",
        headers: {
          "Content-type": "application/json",
        },
        body: JSON.stringify(task),
      });

      const data = await res.json();
      this.tasks = [...this.tasks, data];
    },

    async deleteTask(id) {
      if (confirm("Are you sure?")) {
        const res = await fetch(`api/tasks/${id}`, {
          method: "DELETE",
        });

        res.status === 200
          ? (this.tasks = this.tasks.filter(function (task) {
              return task.id !== id;
            }))
          : alert(`error deleting task ${id}`);
      }
    },

    async toggleReminder(id) {
      const taskToToggle = await this.fetchTask(id);
      const updateTask = { ...taskToToggle, reminder: !taskToToggle.reminder };

      const res = await fetch(`api/tasks/${id}`, {
        method: "PUT",
        headers: {
          "Content-type": "application/json",
        },
        body: JSON.stringify(updateTask),
      });

      const data = await res.json();
      this.tasks = this.tasks.map(function (task) {
        return task.id === id ? { ...task, reminder: data.reminder } : task;
      });
    },
    async created() {
      this.tasks = await this.fetchTasks();
    },
  },
};
</script>
