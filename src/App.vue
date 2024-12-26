<template>
  <main>
    <div>
      <h2>Список дел</h2>
      <form @submit.prevent="editingItem ? saveItem() : addItem()">
        <input
          type="text"
          v-model="title"
          placeholder="Что вам нужно сделать?"
        />
        <input type="text" v-model="user" placeholder="Введите логин" />
        <button type="submit">{{ editingItem ? "Изменить элемент" : "Добавить" }}</button>
        <button v-if="editingItem" type="button" @click="cancelEdit">Отмена</button>
      </form>
    </div>
    <div>
      <h2>Список дел</h2>
      <div class="list-item" v-for="(el, i) in items" :key="el._id">
        <p>
          {{ el.title }} <span>{{ el.user }}</span>
        </p>
        <p>
          {{ new Date(el.published).toLocaleString() }} 
          <button @click="deleteItem(el, i)">Удалить</button>
          <button @click="editItem(el)">Изменить</button>
        </p>
      </div>
    </div>
  </main>
</template>

<script>
import axios from "axios";

export default {
  name: "App",
  data() {
    return {
      items: [],
      title: "",
      user: "",
      editingItem: null,
    };
  },
  async mounted() {
    const result = await axios.get("api/todo-item");
    this.items = result.data;
  },
  methods: {
    async deleteItem(el, i) {
      await axios.delete("api/todo-item/" + el._id);
      this.items.splice(i, 1);
    },
    async addItem() {
      const url = `api/todo-item/?title=${this.title}&user=${this.user}&published=${new Date().toISOString()}`;
      const response = await axios.post(url);
      this.items.push(response.data);
      this.title = "";
      this.user = "";
    },
    editItem(el) {
      this.editingItem = el;
      this.title = el.title;
      this.user = el.user;
    },
    async saveItem() {
      try {
        const url = `api/todo-item/${this.editingItem._id}?title=${this.title}&user=${this.user}&published=${this.editingItem.published}`; 
        const response = await axios.put(url); 
        console.log('Ответ от сервера при редактировании:', response.data); 
        const index = this.items.findIndex(item => item._id === this.editingItem._id); 
        if (index !== -1) { this.items[index].title = this.title; this.items[index].user = this.user; }
        this.cancelEdit();
      } catch (err) {
        console.error('Ошибка при редактировании элемента:', err);
      }
    },
    cancelEdit() { 
      this.editingItem = null; 
      this.title = null;
      this.user = null;
    } 
  },
};
</script>

<style>
.list-item {
  width: 100%;
  background: #dfbc4c;
  padding: 10px 15px;
  margin-top: 20px;
  border-radius: 5px;
  font-size: 14px;
}

.list-item p {
  margin-bottom: 5px;
}

.list-item p span {
  float: right;
  color: #8f772d;
  font-size: 12px;
}

.list-item p button {
  float: right;
  font-size: 9px;
  margin: 0;
  padding: 5px 6px;
}

.list-item p:last-child {
  color: #b1953c;
  font-size: 11px;
}
</style>
