<template>
  <div class="container">
    <div class="input-wrapper">
      <input v-model="description" @keyup.enter="addItem" type="text" /><br />
      <button class="add" @click="addItem()">Add Item</button>
    </div>
    <div class="input-wrapper">
      <input type="search" v-model="search">
    </div>
    <div class="item-container" v-for="todo, index of filteredData" :key="todo.id">
      <div class="todo-wrapper">
        <input type="checkbox" :checked="todo.isDone" @change="setDone(todo.id)">
        <div class="info-wrapper">
          <p class="description">
            {{ todo.description }}
          </p>
          <div class="tags-wrapper">
            <div class="tag" v-for="tag, index of todo.tags" :key="index">{{ tag }}</div>
            <button class="add" @click="openModal(index)">+</button>
          </div>
        </div>
      </div>
      <button class="remove" @click="removeItem(todo.id)">remove</button>
    </div>
    <Modal v-model:visible="isModalVisible" :okButton="{ text: 'save', onclick: saveTags }" title="Choose tags">
      <div class="tags-wrapper">
        <button class="option" :class="{ isActive: tags[index].active }" v-for="tag, index in tags" :key="index"
          @click="toggleTag(index)">{{ tag.name
          }}</button>
      </div>
    </Modal>
  </div>
</template>

<script>
import axios from "axios";
import { Modal } from 'usemodal-vue3';

export default {
  name: 'TodoList',
  components: {
    Modal,
  },
  data() {
    return {
      description: "", //todo description
      search: "", //todo description
      todos: [],
      tags: [{ name: 'tag1', active: false }, { name: 'tag2', active: false }, { name: 'tag3', active: false }],
      isModalVisible: false, // modal state
      todoId: null
    };
  },
  async created() {
    try {
      const res = await axios.get(`http://localhost:3000/todos`);
      this.todos = res.data;
    } catch (error) {
      console.log(error);
    }
  },
  computed: {
    filteredData() {
      return this.todos
        .filter(
          ({ description, tags }) => {
            if (description.toLowerCase().includes(this.search)) {
              return true;
            }

            if (tags.join('').toLowerCase().includes(this.search)) {
              return true;
            }

            return false;
          }
        );
    }
  },

  methods: {
    async setDone(id) {
      const currentStatus = this.todos.find((item) => item.id === id).isDone

      await axios.patch(`http://localhost:3000/todos/${id}`, {
        isDone: !currentStatus,
      });
      this.todos = this.todos.map((item) => {
        if (item.id === id) {
          item.isDone = !currentStatus;
        }
        return item;
      });
    },
    async saveTags() {
      await axios.patch(`http://localhost:3000/todos/${this.todoId}`, {
        tags: this.tags.filter(tag => tag.active).map(tag => tag.name),
      });
      this.todos.map((item) => {
        if (item.id === this.todoId) {
          item.tags = this.tags.filter(tag => tag.active).map(tag => tag.name);
        }
        return item;
      });

      this.closeModal();
    },
    removeItem(id) {
      axios.delete(`http://localhost:3000/todos/${id}`);
      this.todos = this.todos.filter((item) => item.id !== id);
    },
    async addItem() {
      const res = await axios.post(`http://localhost:3000/todos`, {
        description: this.description,
        isDone: false
      });
      this.todos = [...this.todos, res.data];
      this.description = "";
    },
    openModal(index) {
      const selectedTodo = this.todos[index];
      this.isModalVisible = true
      this.todoId = selectedTodo.id;

      this.tags = this.tags.map(({ name }) => ({
        name,
        active: selectedTodo.tags.includes(name) ?? false,
      }));
    },
    closeModal() {
      this.isModalVisible = false
      this.todoId = null
    },
    toggleTag(index) {
      this.tags[index].active = !this.tags[index].active;
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.container {
  display: flex;
  flex-direction: column;
  width: 600px;
  padding: 15px;
  margin: 0 auto;
  border-radius: 10px;
  box-shadow: 0px 0px 20px 3px rgba(0, 0, 0, 0.5);
}

.input-wrapper {
  display: flex;
  justify-content: center;
  gap: 14px;
  margin-bottom: 30px;
}


.item-container {
  display: flex;
  align-items: center;
  padding: 10px;
  margin-bottom: 10px;
  gap: 20px;
  border-radius: 5px;
  box-shadow: 0px 0px 10px 1px rgba(0, 0, 0, 0.2);
}

button {
  height: min-content;
  border-radius: 5px;
  background-color: transparent;
  cursor: pointer;
}

.add {
  border: 1px solid rgb(65, 184, 131);
}

.add:hover {
  box-shadow: 0px 0px 5px 1px rgba(65, 184, 131, 0.5);
}

.remove {
  margin-left: auto;
  border: 1px solid rgb(248, 0, 17);
}

.remove:hover {
  box-shadow: 0px 0px 5px 1px rgba(248, 0, 17, 0.5);
}

.todo-wrapper {
  display: flex;
  align-items: center;
  gap: 10px;
}

.info-wrapper {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.description {
  margin: 0;
}

.tags-wrapper {
  display: flex;
  align-items: center;
  gap: 10px;
}

.tag {
  padding: 0 5px;
  border: 1px solid black;
  border-radius: 5px;
  background-color: lightslategray;

}

.option.isActive {
  background-color: green;
  color: white;
}
</style>
