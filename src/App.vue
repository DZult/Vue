<template>
  <div class="app">
    <h1>Страница с постами</h1>
    <my-input v-model="searchQuery"
              placeholder="Поиск..."
    />
    <div class="app__btns">
      <my-button @click="showDialog">
        Создать пост
      </my-button>
      <my-select v-model="selectedSort"
                 :options="sortOptions"
      />
    </div>
    <my-dialog v-model:show="dialogVisible">
      <post-form @create="createPost"/>
    </my-dialog>
    <post-list :posts="sortedAndSearchedPosts"
               @remove="removePost"
               v-if="!isPostsLoading"
    />
    <div v-else>Идет загрузка...</div>
    <div ref="observer" class="observer"></div>
<!--    <div class="page__wrapper">-->
<!--      <div v-for="pageNumber in totalPages"-->
<!--           :key="pageNumber"-->
<!--           class="page"-->
<!--           :class="{-->
<!--             'current-page' : page === pageNumber-->
<!--           }"-->
<!--           @click="changePage(pageNumber)"-->
<!--      >-->
<!--        {{pageNumber}}-->
<!--      </div>-->
<!--    </div>-->
  </div>
</template>

<script>
import axios from 'axios'
import PostForm from "@/components/PostForm";
import PostList from "@/components/PostList";
import MyDialog from "@/components/UI/MyDialog";
import MySelect from "@/components/UI/MySelect";
import MyInput from "@/components/UI/MyInput";
export default {
    components: {
      MyInput,
      MySelect,
      MyDialog,
      PostList, PostForm
    },
    data() {
      return {
        posts: [],
        dialogVisible: false,
        isPostsLoading: false,
        searchQuery: '',
        selectedSort: '',
        page: 1,
        limit: 10,
        totalPages: 0,
        sortOptions: [
          {value: 'title', name: 'По названию'},
          {value: 'body', name: 'По содержанию'},
        ],
      }
    },
    methods: {
      createPost(post) {
        this.posts.push(post);
        this.dialogVisible = false;
      },
      removePost(post) {
        this.posts = this.posts.filter(p => p.id !== post.id);
      },
      showDialog() {
        this.dialogVisible = true
      },
      async loadMorePosts() {
        try {
          this.isPostsLoading = true;
          setTimeout(async () => {
            const response = await axios.get('https://jsonplaceholder.typicode.com/posts', {
              params: {
                _page: this.page,
                _limit: this.limit,
              }
            });
            this.totalPages = Math.ceil(response.headers['x-total-count'] / this.limit)
            this.posts = [...this.posts, ...response.data];
            this.isPostsLoading = false;
          }, 0);
        } catch (e) {
          alert('Ошибка');
        } finally {
          this.isPostsLoading = false;
        }
      },
      // changePage(pageNumber) {
      //   this.page = pageNumber;
      // }
    },
    mounted() {
      this.loadMorePosts();
      const options = {
        rootMargin: '0px',
        threshold: 1.0
      }
      const callback = function (entries, observer) {

      };
      const observer = new IntersectionObserver(callback, options);
    },
    watch: {
      // page() {
      //   this.fetchPosts();
      // }
    },
    computed: {
      sortedPosts() {
        return [...this.posts].sort((post1, post2) =>  post1[this.selectedSort]?.localeCompare(post2[this.selectedSort]));
      },
      sortedAndSearchedPosts() {
        return this.sortedPosts.filter(post => post.title.toLowerCase().includes(this.searchQuery.toLowerCase()));
      }
    },
  }
</script>

<style>
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  .app {
    padding: 20px;
  }

  .app__btns {
    margin: 15px 0;
    display: flex;
    justify-content: space-between;
  }

  .page__wrapper {
    display: flex;
    margin-top: 15px;
  }

  .page {
    border: 1px solid black;
    padding: 10px;
  }

  .current-page {
    border: 2px solid teal;
  }
  .observer {
    height: 30px;
    background: green;
  }

</style>
