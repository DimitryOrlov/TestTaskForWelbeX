<template>
   <div class="app">
      <h1>Table</h1>
      <section class="select__section">
         <select v-model="column">
            <option disabled value="">Выберите из списка</option>
            <option v-for="option in columnSelect" :value="option[0]">{{option[1]}}</option>
            <!-- <option v-for="option in options" :key="option.value" :value="option.value">{{option.name}}</option> -->
         </select>
         <select v-model="condition">
            <option disabled value="">Выберите из списка</option>
            <option v-for="option in conditionSelect" :value="option[0]">{{option[1]}}</option>
            <!-- <option v-for="option in options" :key="option.value" :value="option.value">{{option.name}}</option> -->
         </select>
         <input type="text" v-model="searchString" placeholder="Поиск">
      </section>
      <table class="table table-hover">
         <thead class="thead-dark">
            <tr>
               <th scope="col">Date</th>
               <th scope="col">Name</th>
               <th scope="col">Count</th>
               <th scope="col">Distance</th>
            </tr>
         </thead>
         <!-- <tbody> -->
         <TransitionGroup name="table-body" tag="tbody" :css="false" @before-enter="onBeforeEnter" @enter="onEnter"
            @leave="onLeave">
            <tr v-for="(post, index) in filteredTable" :key="index">
               <td>{{post.date}}</td>
               <td>{{post.name}}</td>
               <td>{{post.count}}</td>
               <td>{{post.distance}}</td>
            </tr>
         </TransitionGroup>
         <!-- </tbody> -->
      </table>
      <section class="page__section-wrapper">
         <div v-for="pageNumber in totalPages" :key="pageNumber" class="page" :class="{
            'current-page': pageNumber === page 
         }" @click="changePage(pageNumber)">
            {{pageNumber}}
         </div>
      </section>
   </div>
</template>

<script>
import axios from 'axios';

const BASE_URL = `https://jsonplaceholder.typicode.com/posts`;

export default {
   name: 'App',

   components: {

   },

   data() {
      return {
         page: 1,
         limit: 10,
         posts: [
            // array with posts
         ],
         column: '',
         operation: '',
         filterVal: '',
         totalPages: 0,
         isPostsLoading: true,
         column: '',
         columnSelect: [
            ['name', 'имя', 'string'],
            ['count', 'кол-во', 'number'],
            ['distance', 'расстояние', 'number'],
         ],
         condition: '',
         conditionSelect: [
            ['equal', 'равно'],
            ['contains', 'содержит'],
            ['greater', 'больше'],
            ['less', 'меньше'],
         ],
         searchString: '',
         types: {
            string: {
               equal: (a, b) => a.toLowerCase() === b,
               contains: (a, b) => a.toLowerCase().includes(b),
               greater: (a, b) => a.toLowerCase() > b,
               less: (a, b) => a.toLowerCase() < b,
            },
            number: {
               equal: (a, b) => a === +b,
               contains: (a, b) => `${a}`.includes(b),
               greater: (a, b) => a > +b,
               less: (a, b) => a < +b,
            },
         },
      }
   },

   // lifecycle methods
   async created() {
      console.clear();
      console.log('Created');
      await this.fetchPosts();
   },

   mounted() {
      console.log('Mounted');
   },

   onUpdated() {
      console.log('Updated');
   },
   watch: {
      page() {
         this.fetchPosts();
      }
   },
   computed: {
      filteredTable() {
         const { posts, column } = this;
         const type = this.columnSelect.find(n => n[0] === column)?.[2];
         const filterFn = this.types[type]?.[this.condition];
         const searchString = this.searchString.toLowerCase();

         return filterFn && searchString
            ? posts.filter(n => filterFn(n[column], searchString))
            : posts;
      },
   },

   methods: {
      async fetchPosts() {
         try {
            setTimeout(async () => {
               const res = await axios.get(BASE_URL, {
                  params: {
                     _page: this.page,
                     _limit: this.limit,
                  }
               });
               this.posts = res.data.map((elem) => {
                  // random date
                  return { ...elem, name: elem.title.slice(0, 10), date: new Date(Math.floor(Math.random() * Date.now())).toLocaleDateString(), count: Math.floor(Math.random() * 100), distance: Math.floor(Math.random() * 10000) }
               });
               this.totalPages = Math.ceil(res.headers['x-total-count'] / this.limit);
               this.isPostsLoading = false;
            }, 5)
         } catch (e) {
            throw new Error(e);
         } finally {
            this.isPostsLoading = false;
         }
      },
      changePage(pageNumber) {
         this.page = pageNumber;
         this.column = '';
         this.condition = '';
         this.searchString = '';
      },
      onBeforeEnter(element) {
         element.style.height = 0;
         element.style.opacity = 0;
      },
      onEnter(element, done) {
         gsap.to(element, {
            fontSize: "1em",
            height: "1.6em",
            opacity: 1,
            onComplete: done,
         });
      },
      onLeave(element, done) {
         gsap.to(element, {
            opacity: 0,
            onComplete: done,
         });
      },
   }
}
</script>

<style scoped>
.select__section {
   margin: 20px 0 10px 0;
}

.page__section-wrapper {
   display: flex;
   margin-top: 15px;
}

.page {
   border: 1px solid black;
   padding: 10px;
   cursor: pointer;
}

.page:hover {
   background-color: #2c3e50;
   color: aliceblue;
}

.current-page {
   background-color: #2c3e50;
   color: aliceblue;
   /* border: 2px solid #2b6e62; */
}
</style>