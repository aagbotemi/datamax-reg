<template>
  <div class="home">
    <h1>DataMax Registrar Project</h1>
    <div>
      <label for="search">Search: </label>
      <input type="text" name="search" v-model="searchQuery">
    </div>

    <table>
      <thead>
        <tr>
          <th class="text-left">S/N</th>
          <th class="text-left">Name</th>
          <th class="text-left">ISBN</th>
          <th class="text-left">Authors</th>
          <th class="text-left">Pages</th>
          <th class="text-left">Country</th>
          <th class="text-left">Released</th>
        </tr>
      </thead>
      <tbody>
        <tr v-if="loading">
          <td colspan="7" style="text-align: center">Loading...</td>
        </tr>
        <tr v-else-if="filteredBooks.lenght === 0">
          <td colspan="7" style="text-align: center">No book was found</td>
        </tr>
        <tr
          v-for="(book, index) in filteredBooks"
          :key="book.name"
          v-else
        >
          <td>{{ index + 1 }}</td>
          <td>{{ book.name }}</td>
          <td>{{ book.isbn }}</td>
          <td>{{ book.authors.join(", ") }}</td>
          <td>{{ book.numberOfPages }}</td>
          <td>{{ book.country }}</td>
          <td>{{ book.released | formatDate }}</td>
        </tr>
      </tbody>
    </table>
    <v-pagination
        v-model="page"
        :length="2"
        circle
    ></v-pagination>
  </div>
</template>



<script>
import axios from 'axios'

export default {
    components: {
        // HelloWorld,
    },

    data: () => ({
        books: [],
        searchQuery: "",
        loading: false,
        page: 1,
        pageSize: 4,
        totalPages: 0,
    }),
    mounted () {
        this.fetchBooks()
    },
    methods: {
        async fetchBooks () {
        this.loading = true
        try {
            const result = await axios.get(`https://www.anapioficeandfire.com/api/books?page=${this.page}&pageSize=${this.pageSize}`)
            this.books = result.data
            this.page = result.page
            this.books = result.data
            console.log(result);
            this.loading = false
        } catch (error) {
            alert(error.message)
            this.loading = false
        }
        }
    },
    computed: {
        filteredBooks() {
        const query = this.searchQuery.toLowerCase();
        if (this.searchQuery === "") {
            return this.books
        }

        return this.books.filter(book => {
            return Object.values(book).some(name => String(name).toLowerCase().includes(query))
        })
        }
    }
}
</script>



<style>
.home {
  max-width: 750px;
  width: 100%;
  margin: 0 auto;
}
.home h1 {
  font-size: 30px;
  text-align: center;
  font-family: cursive;
}
.home div {
  text-align: right;
  margin-bottom: 3px;
}
input[type="text"] {
  padding: 6px 8px;
  margin-left: auto;
  border: 1px solid #222;
  border-radius: 4px;
}
table {
  font-family: Helvetica, sans-serif;
  border-collapse: collapse;
  width: 100%;
}
th {
  width: 250px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
td, th {
  border: 1px solid #dddddd;
  text-align: left;
  padding: 10px;
}

tr:nth-child(even) {
  background-color: #dddddd;
}
</style>
