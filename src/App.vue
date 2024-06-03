<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input"/><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="submit" class="button">Save</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <h2>{{ article.title }}</h2>
        <p>{{ article.content }}</p>
        <button @click="edit(article)" class="button">Edit</button>
        <button @click="remove(article.id)" class="button delete">Delete</button>
      </li>
    </ul>
    <button @click="load" class="button load">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: ''
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id 
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, form);

        if (form.id) {
          const index = articles.value.findIndex(article => article.id === form.id);
          if (index !== -1) articles.value[index] = response.data;
        } else {
          articles.value.push(response.data);
        }

        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, remove, load };
  }
};
</script>

<style scoped>
.container {
  background-color: #ffe6f0;
  padding: 20px;
  border-radius: 10px;
}

.form {
  margin-bottom: 20px;
}

.input, .textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ff69b4;
  border-radius: 5px;
}

.button {
  background-color: #ff69b4;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin-right: 5px;
}

.button:hover {
  background-color: #ff1493;
}

.button.delete {
  background-color: #ff4444;
}

.button.delete:hover {
  background-color: #cc0000;
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  background-color: #fff0f5;
  padding: 15px;
  margin-bottom: 10px;
  border-radius: 5px;
  border: 1px solid #ff69b4;
}

.load {
  display: block;
  margin: 20px auto;
}
</style>
