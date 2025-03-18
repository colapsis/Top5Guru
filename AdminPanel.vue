<template>
  <div class="admin-panel">
    <h1>Create New Post</h1>
    
    <input v-model="title" @input="updateSlug" placeholder="Enter title" class="input" />
    
    <label>Slug (Auto-generated, cannot be changed)</label>
    <input v-model="slug" class="input" readonly />
    
    <label>Short Description</label>
    <quill-editor ref="shortDescEditor" v-model="shortDescription" theme="snow"></quill-editor>
    
    <label>Long Description</label>
    <quill-editor ref="longDescEditor" v-model="longDescription" theme="snow"></quill-editor>
    
    <label>Tag</label>
    <select v-model="tag" class="input">
      <option value="Trending">Trending</option>
      <option value="Popular">Popular</option>
      <option value="Latest">Latest</option>
      <option value="Random">Random</option>
    </select>
    
    <button @click="submitPost" class="btn">Post</button>
  </div>
</template>

<script>
import { db } from '../firebaseConfig';
import { collection, addDoc } from 'firebase/firestore';
import { QuillEditor } from '@vueup/vue-quill';
import '@vueup/vue-quill/dist/vue-quill.snow.css';
import { useRouter } from 'vue-router';

export default {
  components: {
    QuillEditor
  },
  data() {
    return {
      title: '',
      slug: '',
      shortDescription: '',
      longDescription: '',
      tag: 'Trending'
    };
  },
  setup() {
    const router = useRouter();
    return { router };
  },
  methods: {
    updateSlug() {
      if (this.title) {
        this.slug = this.title
          .toLowerCase()
          .replace(/[^a-z0-9]+/g, '-')
          .replace(/(^-|-$)/g, '');
      }
    },
    getQuillHtml(refName) {
      return this.$refs[refName]?.getHTML() || '';
    },
    async submitPost() {
      try {
        const postData = {
          title: this.title,
          slug: this.slug,
          shortDescription: this.getQuillHtml('shortDescEditor'),
          longDescription: this.getQuillHtml('longDescEditor'),
          tag: this.tag,
          createdAt: new Date()
        };
        console.log("🔍 Before saving to Firestore:", postData);
        await addDoc(collection(db, 'posts'), postData);
        alert('✅ Post successfully added!');
        this.router.push(`/article/${this.slug}`);
      } catch (error) {
        console.error('❌ Error adding post:', error);
      }
    }
  }
};
</script>

<style>
.admin-panel {
  max-width: 600px;
  margin: auto;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.input {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}
.btn {
  background-color: #007bff;
  color: white;
  padding: 10px;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}
.btn:hover {
  background-color: #0056b3;
}
</style>
