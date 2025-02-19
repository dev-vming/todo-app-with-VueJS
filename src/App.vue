<script>
export default{
  // 초기에 넣을 데이터 작성
  data() {
    return {
      todos: JSON.parse(localStorage.getItem("todos")) || [],
      editingTodo: null,
      editingTitle: "",
      visiblilityFilter: "ALL",
    };
  },
  // 데이터가 변경될 때 마다 무엇을 할 것인지 작성
  watch: {
    todos: {
      handler(prev, curr) {
        localStorage.setItem("todos", JSON.stringify(curr));
      },
      deep: true,
    },
  },
  // 특정 데이터가 변경될 떄마다 다른 데이터를 계산하여 반환
  computed: {
    num_remaining_todos() {
      return this.todos.filter((todo) => !todo.completed).length;
    },
    filteredTodos() {
      switch (this.visiblilityFilter) {
        case "ACTIVE":
          return this.todos.filter((todo) => !todo.completed);
        case "COMPLETED":
          return this.todos.filter((todo) => todo.completed);
        default:
          return this.todos;
      }
    },
  },
  // 지시자를 사용하여 특정 돔에 직접적인 접근을 할 수 있게 해줌
  directives: {
    hasFocus: {
      mounted(element, value) {
        if (value) element.focus();
      },
    },
  },
  methods: {
    // 1. 할 일 등록, 2. 할 일 목록 표시 - 엔터 키를 눌렀을 때, 할 일이 목록으로 표시되는 기능
    할일추가(event) {
      if (event.isComposing) return true;

      const title = event.target.value;
      this.todos.push({ title, completed: false });

      event.target.value = "";
    },
    // 3. 할 일 완료 - 할 일의 완료 상태를 표시 및 변경할 수 있는 기능
    할일완료토글(todo) {
      todo.completed = !todo.completed;
    },
    // 5. 할 일 삭제 - 목록에서 특정 할 일을 삭제하는 기능
    할일삭제(idx) {
      this.todos.splice(idx, 1);
    },
    // 6.할 일 수정 - 이미 입력된 할 일의 내용을 수정하는 기능
    할일수정(todo) {
      this.editingTodo = todo;
      this.editingTitle = todo.title;
    },

    할일제목수정(idx, event) {
      if (event.isComposing) return true;
      this.todos[idx].title = this.editingTitle;
      this.editingTodo = null;
      this.editingTitle = "";
    },

    할일수정취소() {
      this.editingTodo = null;
      this.editingTitle = "";
    },
    // 7. 할 일 필터링 - 완료된 할 일과 진행 중인 할 일을 구분하여 볼 수 있는 필터 기능
    전체보기() {
      this.visiblilityFilter = "ALL";
    },

    해야할일보기() {
      this.visiblilityFilter = "ACTIVE";
    },

    완료한일보기() {
      this.visiblilityFilter = "COMPLETED";
    },
    // 8. 할 일 일괄 완료 처리 - 모든 할 일을 한 번에 완료 처리할 수 있는 기능
    전체완료토글() {
      const isAllCompleted = this.todos.every((todo) => todo.completed);
      this.todos = this.todos.map((todo) => ({
        ...todo,
        completed: !isAllCompleted,
      }));
    },

    // 9. 할 일 일괄 삭제 - 완료된 할 일만을 선택적으로 일괄 삭제하는 기능
    완료된할일삭제() {
      this.todos = this.todos.filter((todo) => !todo.completed);
    },
  },
};
</script>

<template>
  <div id="main">
    <div id="todo-app" class="todo-app">
      <div id="num-remaining-todos">남은 할일: {{ num_remaining_todos }}</div>
      <div class="filter-buttons">
        <button
          @click="전체보기()"
          :class="{ active: visiblilityFilter === 'ALL' }"
        >
          전체 보기
        </button>
        <button
          @click="해야할일보기()"
          :class="{ active: visiblilityFilter === 'ACTIVE' }"
        >
          해야할 일 보기
        </button>
        <button
          @click="완료한일보기()"
          :class="{ active: visiblilityFilter === 'COMPLETED' }"
        >
          완료한 일 보기
        </button>
      </div>
      <input
        id="todo-input"
        @keydown.enter="할일추가($event)"
        type="text"
        placeholder="할일을 입력해주세요."
      />
      <ul id="todo-list">
        <li
          v-for="(todo, idx) in filteredTodos"
          :class="[
            'todo',
            { completed: todo.completed, editing: todo === editingTodo },
          ]"
        >
          <template v-if="todo !== editingTodo">
            <input
              type="checkbox"
              :checked="todo.completed"
              @click="할일완료토글(todo)"
            />
            <span @dblclick="할일수정(todo)">{{ todo.title }}</span>
            <button class="del" @click="할일삭제(idx)">
              <i class="fas fa-trash-alt"></i>
            </button>
          </template>
          <template v-else>
            <input
              type="text"
              class="todo-edit-input"
              v-model="editingTitle"
              @keydown.enter="할일제목수정(idx, $event)"
              @blur="할일수정취소()"
              v-has-focus="todo === editingTodo"
            />
          </template>
        </li>
      </ul>

      <div class="action-buttons">
        <button @click="전체완료토글()">전체 완료 토글</button>
        <button @click="완료된할일삭제()">완료한 일 삭제</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
#main {
  background: linear-gradient(135deg, #f5f7fa, #c3cfe2);
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  margin: 0;
}
.todo-app {
  font-family: "Arial", sans-serif;
  max-width: 600px;
  width: 100%;
  margin: 0 auto;
  padding: 20px;
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
}

#todo-input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  margin-bottom: 10px;
  font-size: 16px;
  box-sizing: border-box;
}

#todo-list {
  list-style: none;
  padding: 0;
  width: 100%;
}

.todo {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px;
  border-bottom: 1px solid #eaeaea;
}

.todo.completed {
  text-decoration: line-through;
  color: #999;
}

.todo.editing > * {
  display: none;
}

.todo.editing .todo-edit-input {
  display: block;
}

.todo-edit-input {
  display: none;
  flex: 1;
  margin-left: 10px;
  padding: 5px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 8px 12px;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #0056b3;
}

button.del {
  background-color: #ff4d4d;
  text-decoration: none;
  color: white;
  border: none;
  padding: 8px;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background-color 0.3s;
}

button.del:hover {
  background-color: #cc0000;
}

input[type="checkbox"] {
  width: 20px;
  height: 20px;
  border: 1px solid #ccc;
  cursor: pointer;
  transition: background-color 0.3s;
}

input[type="checkbox"]:checked {
  background-color: #007bff;
  border-color: #007bff;
}

#num-remaining-todos {
  font-weight: bold;
  position: absolute;
  top: 20px;
  left: 20px;
}

.filter-buttons {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 10px;
  width: 100%;
}

.filter-buttons button {
  background-color: transparent;
  color: #333;
  border: none;
  padding: 8px 12px;
  cursor: pointer;
  transition: background-color 0.3s, font-weight 0.3s;
}

.filter-buttons button:hover {
  background-color: #e1e1e1;
  font-weight: bold;
}

.filter-buttons button.active {
  background-color: #434343;
  color: white;
  border-bottom: none;
}

.action-buttons {
  display: flex;
  justify-content: space-between;
  width: 100%;
  margin-top: 10px;
}

.action-buttons button {
  flex: 1;
  margin: 0 5px;
}
</style>
