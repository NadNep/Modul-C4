<template>
  <div class="container">
    <h1>Todos</h1>
    <div class="alert alert-success alert-dismissible fade show" role="alert">
      Бутстрап подключен и работает
      <button type="button" class="close" data-dismiss="alert" aria-label="Close">
        <span aria-hidden="true">&times;</span>
      </button>
    </div>
    <table class="table">
      <thead>
        <tr>
          <th>Uid</th>
          <th>Текст задачи</th>
          <th>Выполнена?</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(todo) in todos">
          <td>{{ todo[0] }}</td>
          <td class="text-center">{{ JSON.parse(todo[1]).task }}</td>
          <td class="text-center">
              <span v-if="JSON.parse(todo[1]).status=='true'">Выполнено</span>
              <span v-else>Не выполнено</span>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>

export default {
  name: 'Fetch',
  data() {
    return {
      todos: [],
    };
  },
  methods: {
    getTodos() {
      var all_storage = Object.entries(localStorage);
      var task_storage = [];
      all_storage.forEach(function(element) {
        if (element[0].includes("task")) {
          element[0] = element[0].split("_")[1]
          task_storage.push(element)
        }
      });
      this.todos = task_storage;
    },
  },
  created() {
    this.getTodos();
  },
};
</script>
