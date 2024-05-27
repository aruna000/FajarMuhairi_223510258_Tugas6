<template>
  <div>
    <form @submit.prevent="save">
      <input type="text" v-model="form.title" placeholder="Title" /><br />
      <textarea v-model="form.content" placeholder="Content"></textarea><br />
      <button type="submit">Save</button>
    </form>
    <ul>
      <li v-for="article in articles" :key="article.id">
        <h3>{{ article.title }}</h3>
        <p>{{ article.content }}</p>
        <button @click="edit(article)">Edit</button>
        <button @click="remove(article.id)">Delete</button>
      </li>
    </ul>
    <button @click="load">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from "vue";
import axios from "axios";

export default {
  setup() {
    const form = reactive({
      id: null,
      title: "",
      content: "",
    });
    const articles = ref([]);
    const apiUrl = "http://localhost:3000/articles";

    async function load() {
      try {
        const response = await axios.get(apiUrl);
        articles.value = response.data;
      } catch (error) {
        console.error("Error loading articles:", error);
      }
    }

    async function save() {
      try {
        const url = form.id ? `${apiUrl}/${form.id}` : apiUrl;
        const method = form.id ? "put" : "post";
        const response = await axios[method](url, {
          title: form.title,
          content: form.content,
        });

        if (form.id) {
          const index = articles.value.findIndex(
            (article) => article.id === form.id
          );
          if (index !== -1) {
            articles.value[index] = response.data;
          }
        } else {
          articles.value.push(response.data);
        }

        form.id = null;
        form.title = "";
        form.content = "";
      } catch (error) {
        console.error("Error saving article:", error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`${apiUrl}/${id}`);
        articles.value = articles.value.filter((article) => article.id !== id);
      } catch (error) {
        console.error("Error deleting article:", error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, load, remove };
  },
};
</script>
<style scoped>
div {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

form {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 20px;
}

input[type="text"],
textarea {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
  width: 100%;
  box-sizing: border-box;
}

textarea {
  resize: vertical;
  height: 100px;
}

button[type="submit"] {
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  background-color: #28a745;
  color: #fff;
  font-size: 16px;
  cursor: pointer;
}

button[type="submit"]:hover {
  background-color: #218838;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  background-color: #f8f9fa;
  padding: 15px;
  border: 1px solid #dee2e6;
  border-radius: 4px;
  margin-bottom: 10px;
}

li h3 {
  margin: 0 0 10px 0;
}

li p {
  margin: 0 0 10px 0;
}

button {
  padding: 8px 12px;
  border: none;
  border-radius: 4px;
  font-size: 14px;
  cursor: pointer;
}

button:hover {
  opacity: 0.9;
}

button:nth-of-type(1) {
  background-color: #007bff;
  color: #fff;
  margin-right: 5px;
}

button:nth-of-type(2) {
  background-color: #dc3545;
  color: #fff;
}

button:nth-of-type(2):hover {
  background-color: #c82333;
}

button:nth-of-type(3) {
  background-color: #6c757d;
  color: #fff;
  margin-top: 20px;
}

button:nth-of-type(3):hover {
  background-color: #5a6268;
}
</style>
