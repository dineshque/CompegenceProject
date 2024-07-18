<template>
  <div class="post-details-page">
    <div v-if="loading" class="loading">Loading post details...</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else-if="post">
      <div class="post-header">
        <h1>{{ post.title }}</h1>
        <p class="post-body">{{ post.body }}</p>
      </div>
      <div v-if="user" class="user-details">
        <h2>Author Details</h2>
        <div class="user-info">
          <p><strong>Name:</strong> {{ user.name }}</p>
          <p><strong>Email:</strong> {{ user.email }}</p>
          <p><strong>Website:</strong> <a :href="'http://' + user.website" target="_blank">{{ user.website }}</a></p>
        </div>
      </div>
      <div class="comments-section">
        <h2>Comments</h2>
        <div v-if="comments.length === 0" class="no-comments">No comments yet.</div>
        <div v-else class="comments">
          <div v-for="comment in comments" :key="comment.id" class="comment">
            <h3>{{ comment.name }}</h3>
            <p>{{ comment.body }}</p>
            <p class="email">{{ comment.email }}</p>
          </div>
        </div>
      </div>
    </div>
    <div v-else class="no-post">No post found.</div>
  </div>
</template>

<script>
import { mapState, mapActions } from 'vuex'

export default {
  name: 'PostDetailsPage',
  data() {
    return {
      loading: true,
      error: null
    }
  },
  computed: {
    ...mapState(['post', 'user', 'comments'])
  },
  methods: {
    ...mapActions(['fetchPost', 'fetchUser', 'fetchComments']),
    async loadPostData() {
      this.loading = true
      this.error = null
      try {
        const postId = parseInt(this.$route.params.id)
        await this.fetchPost(postId)
        await this.fetchComments(postId)
        if (this.post) {
          await this.fetchUser(this.post.userId)
        }
      } catch (error) {
        this.error = 'Failed to load post details. Please try again later.'
      } finally {
        this.loading = false
      }
    }
  },
  created() {
    this.loadPostData()
  },
  watch: {
    '$route.params.id': function() {
      this.loadPostData()
    }
  }
}
</script>

<style scoped>
.post-details-page {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Arial', sans-serif;
}

.loading, .error, .no-post, .no-comments {
  text-align: center;
  font-size: 18px;
  color: #555;
}

.post-header {
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.post-header h1 {
  color: #670630;
  margin: 0 0 10px;
}

.post-body {
  color: #555;
}

.user-details {
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.user-details h2 {
  color: #670630;
  margin-bottom: 15px;
}

.user-info p {
  margin: 5px 0;
}

.user-info a {
  color: #3498db;
  text-decoration: none;
}

.user-info a:hover {
  text-decoration: underline;
}

.comments-section {
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.comments-section h2 {
  color: #670630;
  margin-bottom: 15px;
}

.comments .comment {
  background-color: #f9f9f9;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 15px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

.comments .comment h3 {
  margin: 0 0 10px;
}

.comments .comment p {
  margin: 5px 0;
}

.comments .comment .email {
  font-size: 12px;
  color: #7f8c8d;
}
</style>
