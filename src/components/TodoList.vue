<template>
  <div class="container">
    <div class="input-wrapper">
      <input v-model="description" @keyup.enter="addItem" type="text" /><br />
      <button class="add" @click="addItem()">Add Item</button>
    </div>
    <div class="item-container" v-for="todo of todos" :key="todo.id">
      <div class="todo-wrapper">
        <input type="checkbox" :checked="todo.isDone" @change="setDone(todo.id)">
        <div class="info-wrapper">
          <p class="description">
            {{ todo.description }}
          </p>
          <div class="tags-wrapper">
            <div class="tag" v-for="tag, index of todo.tags" :key="index">{{ tag }}</div>
            <button class="add" @click="openModal()">+</button>
          </div>
        </div>
      </div>
      <button class="remove" @click="removeItem(todo.id)">remove</button>
    </div>
    <Modal v-model:visible="isModalVisible" :okButton="{ text: 'save', onclick: null }">
      <div>
        <multiselect v-model="selected" :options="options">
        </multiselect>
      </div>
    </Modal>
  </div>
</template>

<script>
import axios from "axios";
import { Modal } from 'usemodal-vue3';
import Multiselect from 'vue-multiselect';

export default {
  name: 'TodoList',
  components: {
    Multiselect,
    Modal,
  },
  data() {
    return {
      description: "", //todo description
      todos: [],
      isModalVisible: false, // modal state
      tags: [], // todo tags
      selected: null,
      options: ['list', 'of', 'options']
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
    totalTodos() {
      return this.todos.length; //auto increment of 1 of each items added into array
    },
    isComplete() {
      return this.todos.filter(item => item.isDone).length; //to get done 
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
    openModal() {
      this.isModalVisible = true
    },
    async setTags(id) {
      const res = await axios.post(`http://localhost:3000/todos/${id}`, {
        tags: this.tags,
      });
      this.todos.find((item) => item.id === id).tags = [...this.todos, res.data];
      this.tags = [];
    },
  },
}
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>

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
</style>
