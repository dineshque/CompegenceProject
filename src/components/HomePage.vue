<template>
    <nav class="navbar">
      <div class="nav-container">
        <div class="nav-title">
          <h1>Latest Posts</h1>
        </div>
        <div class="nav-search">
          <input type="text" v-model="searchQuery" placeholder="Search posts...">
          <button @click="clearFilters">Clear</button>
        </div>
        <div class="nav-sort">
          <select v-model="sortCriteria">
            <option value="title">Sort by Title</option>
            <option value="author">Sort by Author</option>
          </select>
        </div>
      </div>
    </nav>

    <!-- Loading, Error, and Data Display -->
    <div class="content">
      <div v-if="loading" class="loading">Loading posts...</div>
      <div v-else-if="error" class="error">{{ error }}</div>
      <div v-else-if="filteredPosts.length === 0" class="no-posts">No posts available.</div>
      <div v-else class="posts">
        <div v-for="post in filteredPosts" :key="post.id" class="post" @click="goToPost(post.id)">
          <h2>{{ post.title }}</h2>
          <p>{{ post.body }}</p>
          <p class="author"><strong>Author:</strong> {{ getUserName(post.userId) }}</p>
        </div>
      </div>
    </div>
</template>

<script>
import { mapState, mapActions } from 'vuex'

export default {
  name: 'HomePage',
  data() {
    return {
      loading: true,
      error: null,
      searchQuery: '',
      sortCriteria: 'title'
    }
  },
  computed: {
    ...mapState(['posts', 'users']),
    filteredPosts() {
      let filtered = this.posts;

      if (!Array.isArray(filtered)) {
        return [];
      }

      if (this.searchQuery) {
        filtered = filtered.filter(post =>
          post.title.toLowerCase().includes(this.searchQuery.toLowerCase()) ||
          post.body.toLowerCase().includes(this.searchQuery.toLowerCase())
        );
      }

      return this.sortPosts(filtered);
    }
  },
  methods: {
    ...mapActions(['fetchPosts', 'fetchUsers']),
    getUserName(userId) {
      const user = this.users.find(user => user.id === userId)
      return user ? user.name : 'Unknown'
    },
    goToPost(postId) {
      this.$router.push(`/post/${postId}`)
    },
    sortPosts(posts) {
      if (!Array.isArray(posts)) {
        return [];
      }

      if (this.sortCriteria === 'title') {
        return posts.sort((a, b) => a.title.localeCompare(b.title));
      } else if (this.sortCriteria === 'author') {
        return posts.sort((a, b) => this.getUserName(a.userId).localeCompare(this.getUserName(b.userId)));
      }
      return posts;
    },
    clearFilters() {
      this.searchQuery = '';
      this.sortCriteria = 'title'; // Reset sort criteria to 'title' when clearing filters
    },
    async loadData() {
      this.loading = true
      this.error = null
      try {
        await Promise.all([this.fetchPosts(), this.fetchUsers()])
      } catch (error) {
        this.error = 'Failed to load data. Please try again later.'
      } finally {
        this.loading = false
      }
    }
  },
  created() {
    this.loadData()
  }
}
</script>

<style scoped>
.home-page {
  max-width: 1000px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Arial', sans-serif;
}

.navbar {
  background-color: #1b056e;
  padding: 10px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-radius: 8px;
  margin-bottom: 20px;
}

.nav-container {
  display: flex;
  align-items: center;
  width: 100%;
}

.nav-title h1 {
  color: #fff;
  margin: 0;
}

.nav-search {
  display: flex;
  align-items: center;
  flex: 1;
  margin-left: 20px;
}

.nav-search input[type="text"] {
  padding: 8px;
  border: none;
  border-radius: 4px;
  font-size: 14px;
  flex: 1;
  margin-right: 10px;
}

.nav-search button {
  padding: 8px 16px;
  background-color: #c1c5d4;
  color: rgb(48, 6, 94);
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.nav-search button:hover {
  background-color: #1f5a7b;
}

.nav-sort {
  margin-left: 20px;
}

.nav-sort select {
  padding: 8px;
  border: none;
  border-radius: 4px;
  font-size: 14px;
}

.content {
  margin-top: 20px;
}

.loading, .error, .no-posts {
  text-align: center;
  font-size: 18px;
  color: #555;
}

.posts {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.post {
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  padding: 20px;
  cursor: pointer;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.post:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.15);
}

.post h2 {
  color: #670630;
  margin-top: 0;
}

.post p {
  color: #555;
}

.post .author {
  color: #7f8c8d;
  font-style: italic;
}
</style>
