<template>
  <div class="chuck-norris-container">
    <h1 class="page-title">Chuck Norris Jokes</h1>
    <label class="filter-label">Select a type of filter:</label>
    <nav class="navigation">
      <button @click="toggleCategoryDropdown" class="nav-button">Category</button>
      <button @click="toggleFreeText" class="nav-button">Free Text</button>
    </nav>
    <div v-if="showCategoryDropdown" class="category-dropdown">
      <div class="input-container">
        <select v-model="selectedCategory" class="select-box">
          <option value="" disabled>Select a category</option>
          <option v-for="category in categories" :value="category">{{ category }}</option>
        </select>
        <button @click="fetchJoke('category')" class="action-button">Get Joke</button>
      </div>
    </div>
    <div v-if="showFreeText" class="free-text-input">
      <div class="input-container">
        <input type="text" v-model="freeText" placeholder="Enter free text" class="text-input">
        <button @click="fetchJoke('freeText')" class="action-button">Get Jokes</button>
      </div>
    </div>
    
    <div v-if="jokes.length > 0" class="jokes">
      <div class="joke" v-for="joke in displayedJokes" :key="joke.id">
        <div class="joke-header">
          {{ joke.date }}
        </div>
        {{ joke.value }}
        <!-- Pagination buttons inside the joke box -->
        <div class="pagination" v-if="showPagination">
          <button @click="loadPage(currentPage - 1)" :disabled="currentPage === 1" class="pagination-button">Previous</button>
          <span class="page-number">{{ currentPage }}</span>
          <button @click="loadPage(currentPage + 1)" :disabled="currentPage === totalPages" class="pagination-button">Next</button>
        </div>
      </div>
    </div>
    
    <div v-else-if="jokes.length === 0 && isSearchPerformed" class="no-matches">
      No matching jokes found.
    </div>
  </div>
</template>
<script>
import axios from 'axios';

export default {
  data() {
    return {
      jokes: [],
      showCategoryDropdown: false,
      showFreeText: false,
      selectedCategory: '',
      categories: [],
      freeText: '',
      isSearchPerformed: false,
      currentPage: 1,
      itemsPerPage: 1,
      currentJokeIndex: 0,
      totalPages: 1,
    };
  },
  computed: {
    displayedJokes() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.jokes.slice(start, end);
    },
    showPagination() {
      return this.showFreeText;
    },
  },
  methods: {
    async fetchJoke(type) {
      if (type === 'freeText' && this.freeText) {
        this.currentPage = 1;
        this.fetchJokesWithPagination();
      } else if (type === 'category' && this.selectedCategory) {
        this.fetchCategoryJokes();
      } else {
        this.isSearchPerformed = false;
        this.jokes = [];
        this.totalPages = 1;
      }
    },
    async fetchJokesWithPagination() {
      try {
        const response = await axios.get(
          `https://api.chucknorris.io/jokes/search?query=${this.freeText}`
        );
        const jokes = response.data.result;

        // Add the date property to each joke
        const currentDate = new Date().toLocaleString();
        jokes.forEach(joke => {
          joke.date = currentDate;
        });

        this.jokes = jokes;
        this.totalPages = Math.ceil(jokes.length / this.itemsPerPage);
        this.isSearchPerformed = true;
      } catch (error) {
        console.error('Error fetching Chuck Norris jokes:', error);
        this.jokes = [];
        this.totalPages = 1;
        this.isSearchPerformed = false;
      }
    },
    async fetchCategoryJokes() {
      try {
        const response = await axios.get(
          `https://api.chucknorris.io/jokes/random?category=${this.selectedCategory}`
        );
        const joke = response.data;

        // Add the date property to the single joke
        const currentDate = new Date().toLocaleString();
        joke.date = currentDate;

        this.jokes = [joke];
        this.totalPages = 1;
        this.isSearchPerformed = true;
      } catch (error) {
        console.error('Error fetching Chuck Norris jokes by category:', error);
        this.jokes = [];
        this.totalPages = 1;
        this.isSearchPerformed = false;
      }
    },
    toggleCategoryDropdown() {
      this.showCategoryDropdown = true;
      this.showFreeText = false;
      this.fetchCategories();
    },
    toggleFreeText() {
      this.showCategoryDropdown = false;
      this.showFreeText = true;
    },
    async fetchCategories() {
      try {
        const response = await axios.get('https://api.chucknorris.io/jokes/categories');
        this.categories = response.data;
      } catch (error) {
        console.error('Error fetching categories:', error);
      }
    },
    loadPage(page) {
      if (page >= 1 && page <= this.totalPages) {
        this.currentPage = page;
      }
    },
  },
};
</script>
<style scoped>
body {
  font-family: 'Arial', sans-serif;
  margin: 0;
  padding: 0;
}

.chuck-norris-container {
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 100vh;
  background: linear-gradient(135deg, #3498db, #9b59b6);
  color: white;
}

.filter-label {
  font-size: 20px;
  margin-bottom: 15px;
}

.navigation {
  display: flex;
  justify-content: center;
  gap: 20px;
  margin-bottom: 20px;
}

.nav-button {
  padding: 12px 24px;
  background-color: #e74c3c;
  color: white;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  font-size: 18px;
  transition: background-color 0.3s;
}

.nav-button:hover {
  background-color: #c0392b;
}

.category-dropdown,
.free-text-input {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.select-box,
.text-input {
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ddd;
  border-radius: 5px;
  width: 100%;
  outline: none;
}

.action-button {
  padding: 12px 24px;
  background-color: #e74c3c;
  color: white;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  font-size: 18px;
  transition: background-color 0.3s;
}

.action-button:hover {
  background-color: #c0392b;
}

.page-title {
  font-size: 24px;
  font-weight: bold;
  margin: 20px 0;
}

.jokes {
  margin-top: 20px;
}

.joke {
  font-size: 20px;
  background-color: #f0f0f0;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  max-width: 400px;
  word-wrap: break-word;
  margin-bottom: 10px;
  color:black;
}

.pagination {
  margin-top: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.pagination-button {
  padding: 12px 24px;
  background-color: #e74c3c;
  color: white;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  font-size: 18px;
  transition: background-color 0.3s;
}

.pagination-button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.page-number {
  font-size: 18px;
  margin: 0 10px;
}

.no-matches {
  font-size: 18px;
  color: #FF0000;
  margin-top: 20px;
}

@media (max-width: 768px) {
  .navigation {
    flex-direction: column;
    align-items: center;
  }
  .filter-label {
    text-align: center;
  }
}

.input-container {
  display: flex;
  align-items: center;
  gap: 10px;
}
</style>