<template>
  <div id="app">
    <header>
      <h1>Article Manager</h1>
    </header>
    <main>
      <div class="card">
        <form @submit.prevent="save">
          <input type="text" v-model="form.title" placeholder="Title" class="form-input" /><br />
          <textarea v-model="form.content" placeholder="Content" class="form-textarea"></textarea><br />
          <button type="submit" class="btn-save">Save</button>
        </form>
      </div>
      <ul class="article-list">
        <li v-for="article in articles" :key="article.id" class="article-item">
          <h3>{{ article.title }}</h3>
          <p>{{ article.content }}</p>
          <div class="button-group">
            <button @click="edit(article)" class="btn-edit">Edit</button>
            <button @click="remove(article.id)" class="btn-delete">Delete</button>
          </div>
        </li>
      </ul>
      <button @click="load" class="btn-load">Load Articles</button>
    </main>
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
          articles.value = articles.value.map(article =>
            article.id === response.data.id ? response.data : article
          );
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
}
</script>

<style>
body {
  font-family: Arial, sans-serif;
  background-color: #f5f5f5;
  margin: 0;
  padding: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

#app {
  width: 80%;
  max-width: 800px;
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

header h1 {
  text-align: center;
  color: #333;
}

.card {
  background-color: #fefefe;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.form-input, .form-textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.btn-save, .btn-load {
  background-color: #28a745;
  color: #fff;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
}

.btn-save:hover, .btn-load:hover {
  background-color: #218838;
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  background-color: #fafafa;
  padding: 15px;
  margin-bottom: 10px;
  border: 1px solid #eee;
  border-radius: 4px;
}

.button-group {
  display: flex;
  gap: 10px;
}

.btn-edit, .btn-delete {
  padding: 5px 10px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn-edit {
  background-color: #007bff;
  color: #fff;
}

.btn-edit:hover {
  background-color: #0069d9;
}

.btn-delete {
  background-color: #dc3545;
  color: #fff;
}

.btn-delete:hover {
  background-color: #c82333;
}
</style>
