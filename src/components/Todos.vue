<template>
  <div class="container">
    <div class="col-sm-10">
      <h1>Задачи</h1>
      <table class="table table-bordered">
        <thead>
          <tr>
            <th>Задач выполнено</th>
            <th>Задач не выполнено</th>
            <th>Задач всего</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td class="text-center">{{tasks_done}}</td>
            <td class="text-center">{{tasks_not_done}}</td>
            <td class="text-center">{{tasks_sum}}</td>
          </tr>
        </tbody>
      </table>
      <div :key="rerenderKey">
        <confirmation
          :message="confirmationMessage"
          :SendedAlertStatus="dAlertStatus"
          v-if="showConfirmation">
        </confirmation>
      </div>
      <button type="button"
              id="task-add"
              class="btn btn-success btn-sm align-left d-block"
              v-b-modal.todo-modal>
        Добавить задачу
      </button>
      <table class="table table-dark table-stripped table-hover">
        <thead class="thead-light">
          <tr>
            <th>Uid</th>
            <th>Задача</th>
            <th>Выполнена?</th>
            <th></th>
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
            <td>
              <div class="btn-group" role="group">
                <button type="button"
                        class="btn btn-secondary btn-sm"
                        v-b-modal.todo-update-modal
                        @click="updateTodo(todo)"
                        >
                  Обновить
                </button>
                &nbsp;
                <button type="button"
                        class="btn btn-danger btn-sm"
                        @click="deleteTodo(todo)">
                  X
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <b-modal ref="addTodoModal"
      id="todo-modal"
      title="Добавить задачу"
      hide-footer>
      <b-form @submit="onSubmit" @reset="onReset" class="w-100">
        <b-form-group id="form-description-group"
                  label="Описание:"
                  label-for="form-description-input">
          <b-form-input id="form-description-input"
                  type="text"
                  v-model="addTodoForm.description"
                  required
                  placeholder="Завести задачу"
                  minlength="3"
                  maxlength="56">
          </b-form-input>
        </b-form-group>
        <b-form-group id="form-complete-group">
          <b-form-checkbox-group v-model="addTodoForm.is_completed" id="form-checks">
            <b-form-checkbox value="true">Задача выполнена?</b-form-checkbox>
            </b-form-checkbox-group>
        </b-form-group>
        <b-button type="submit" variant="primary">Добавить</b-button>
        <b-button type="reset" variant="danger">Сброс</b-button>
      </b-form>
    </b-modal>
    <b-modal ref="updateTodoModal"
             id="todo-update-modal"
             title="Update"
             hide-footer>
      <b-form @submit="onUpdateSubmit" @reset="onUpdateReset" class="w-100">
      <b-form-group id="form-update-description-group"
                    label="Описание:"
                    label-for="form-update-description-input">
        <div>
          <h3>Меняем статус задачи {{ modal_task }}</h3>
        </div>
      </b-form-group>
      <b-form-group id="form-update-complete-group">
        <b-form-checkbox-group v-model="updateTodoForm.is_completed" id="form-update-checks">
          <b-form-checkbox value="true">Задача выполнена?</b-form-checkbox>
        </b-form-checkbox-group>
      </b-form-group>
      <b-button-group>
        <b-button type="submit" variant="primary">Обновить</b-button>
        <b-button type="reset" variant="danger">Сброс</b-button>
      </b-button-group>
      </b-form>
    </b-modal>
  </div>
</template>
<style>
  button#task-add {
    margin-top: 20px;
    margin-bottom: 20px;
  }
  h1, td {
    text-align: left;
  }
  .todo-uid {
    text-align: right;
  }
</style>

<script>
import Confirmation from './Confirmation.vue';
const storage_prefix = "task_";
export default {
  name: 'Todos',
  data() {
    return {
      todos: [],
      addTodoForm: [],
      updateTodoForm: [],
      confirmationMessage: '',
      showConfirmation: false,
      dAlertStatus: false,
      rerenderKey: 0,
      tasks_sum: 0,
      tasks_done: 0,
      tasks_not_done: 0,
      next_task_id: 1,
      modal_task: '',
    };
  },
  // end of data
  methods: {
    getTodos() {

      var tasks_todo_counter = 0
      var tasks_done_todo_counter = 0
      var tasks_not_done_todo_counter = 0
      var all_storage = Object.entries(localStorage);
      var task_storage = [];
      all_storage.forEach(function(element) {
        if (element[0].includes("task")) {
          element[0] = element[0].split("_")[1]
          task_storage.push(element)
          tasks_todo_counter += 1
          if (JSON.parse(element[1]).status == 'true') {
            tasks_done_todo_counter += 1
          } else {
            tasks_not_done_todo_counter += 1
          }
        }
      });
      this.tasks_sum = tasks_todo_counter;
      this.tasks_done = tasks_done_todo_counter;
      this.tasks_not_done = tasks_not_done_todo_counter;
      this.todos = task_storage;
    },
    resetForm() {
      this.addTodoForm.description = '';
      this.addTodoForm.is_completed = [];
      this.updateTodoForm.description = '';
      this.updateTodoForm.is_completed = [];
    },
    onSubmit(event) {
      event.preventDefault();
      this.$refs.addTodoModal.hide();
      this.forceRerender();
      this.taskIdMaker()
      // задаю задачу с префиксом и ее статус из данных формы(0 - не выполнена 1 выполнена)
      const task_id = storage_prefix + this.next_task_id;
      var is_completed = this.addTodoForm.is_completed;
      var task_to_add = `{"task": "${this.addTodoForm.description}", "status": "${is_completed}"}`;
      localStorage.setItem(task_id, task_to_add)
      localStorage.setItem('keyCounter', this.next_task_id)
      this.getTodos();
      this.confirmationMessage = `Задача "${this.addTodoForm.description}" добавлена`;
      this.dAlertStatus = true;
      this.showConfirmation = true;
      this.resetForm();
    },
    onReset(event) {
      event.preventDefault();
      this.$refs.addTodoModal.hide();
      this.resetForm();
    },
    updateTodo(todo) {
      this.forceRerender();
      this.updateTodoForm = todo;
      this.modal_task = JSON.parse(todo[1]).task
    },
    onUpdateSubmit(event) {
      event.preventDefault();
      this.$refs.updateTodoModal.hide();
      var task_to_update = `{"task": "${JSON.parse(this.updateTodoForm[1]).task}", "status": "${this.updateTodoForm.is_completed}"}`;
      localStorage.setItem('task_' + this.updateTodoForm[0], task_to_update)
      this.getTodos();
      this.confirmationMessage = `Задача ${JSON.parse(this.updateTodoForm[1]).task} обновлена`;
      this.dAlertStatus = true;
      this.showConfirmation = true;
    },
    onUpdateReset(event) {
      event.preventDefault();
      this.$refs.updateTodoModal.hide();
      this.resetForm();
    },
    deleteTodo(todo) {
      this.forceRerender();
      localStorage.removeItem(`task_${todo[0]}`)
      this.getTodos();
      this.confirmationMessage = `Задача ${todo[0]} удалена из списка`;
      this.dAlertStatus = true;
      this.showConfirmation = true;
    },
    forceRerender() {
      this.showConfirmation = false;
      this.rerenderKey += 1;
    },
    taskIdMaker() {
      if (localStorage.getItem("keyCounter") === null) {
        localStorage.setItem('keyCounter', 1)
      } else {
        this.next_task_id = Number(localStorage.getItem("keyCounter")) + 1;
      }
    },
  },
  components: {
    confirmation: Confirmation,
  },
  created() {
    this.getTodos();
  },
};
</script>