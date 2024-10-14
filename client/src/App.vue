<template>
  <div>
    <h3>Quản lý công việc</h3>
    <div>
      <input v-model="newTask" type="text" placeholder="Nhập tên công việc" />
      <br />
      <button @click="handleClick">Thêm công việc</button>
    </div>
    <div>
      <button @click="filterTasks('all')">Tất cả</button>
      <button @click="filterTasks('completed')">Hoàn thành</button>
      <button @click="filterTasks('in-progress')">Đang thực hiện</button>
    </div>
    <div>
      <ul>
        <li class="todoList" v-for="todo in filteredTodoList" :key="todo.id">
          <input
            type="checkbox"
            v-model="todo.completed"
            @change="updateTask(todo)"
          />
          <p :class="{ completed: todo.completed }">{{ todo.name }}</p>
          <div>
            <i class="fa-solid fa-pen" @click="editTask(todo)"></i>
            <i class="fa-solid fa-delete-left" @click="deleteTask(todo.id)"></i>
          </div>
        </li>
      </ul>
    </div>
    <div>
      <button @click="deleteCompletedTasks">Xóa công việc hoàn thành</button>
      <button @click="deleteAllTasks">Xóa tất cả công việc</button>
    </div>
  </div>
</template>
  
  <script setup>
import axios from "axios";
import { onMounted, ref, computed } from "vue";

const todoList = ref([]);
const newTask = ref("");
const filter = ref("all");

const getAllTodo = async () => {
  const response = await axios.get("http://localhost:3000/todoList");
  todoList.value = response.data;
  console.log("todoList", todoList.value);
};

const handleClick = async () => {
  if (newTask.value.trim()) {
    const newTodo = { name: newTask.value, completed: false };
    const response = await axios.post(
      "http://localhost:3000/todoList",
      newTodo
    );
    todoList.value.push(response.data);
    newTask.value = "";
  }
};

const updateTask = async (todo) => {
  await axios.put(`http://localhost:3000/todoList/${todo.id}`, todo);
};

const deleteTask = async (id) => {
  await axios.delete(`http://localhost:3000/todoList/${id}`);
  todoList.value = todoList.value.filter((todo) => todo.id !== id);
};

const deleteCompletedTasks = async () => {
  for (const todo of todoList.value) {
    if (todo.completed) {
      await axios.delete(`http://localhost:3000/todoList/${todo.id}`);
    }
  }
  todoList.value = todoList.value.filter((todo) => !todo.completed);
};

const deleteAllTasks = async () => {
  for (const todo of todoList.value) {
    await axios.delete(`http://localhost:3000/todoList/${todo.id}`);
  }
  todoList.value = [];
};

const filterTasks = (status) => {
  filter.value = status;
};

const filteredTodoList = computed(() => {
  if (filter.value === "completed") {
    return todoList.value.filter((todo) => todo.completed);
  } else if (filter.value === "in-progress") {
    return todoList.value.filter((todo) => !todo.completed);
  } else {
    return todoList.value;
  }
});

onMounted(() => {
  getAllTodo();
});
</script>
  

<style>
h3 {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 20px;
  text-align: center;
}

div {
  margin-bottom: 20px;
}

input[type="text"] {
  width: 100%;
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #ccc;
  box-sizing: border-box;
  font-size: 16px;
  margin-bottom: 10px;
}

.todoList {
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid #eee;
  padding: 10px 0;
}

.todoList p {
  margin: 0;
  flex-grow: 1;
  padding-left: 10px;
}

.todoList input[type="checkbox"] {
  margin-right: 10px;
}

.todoList div {
  display: flex;
  gap: 10px;
}
</style>