<template>
    <div class="home">
        <h1>DataMax Registrar Project</h1>
        

        <div style="overflow-x:auto; margin:0 2rem;">
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
                    <td colspan="7" style="text-align: center">
                        <loading />
                    </td>
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

        <div> 
            <pagination v-model="page" :records="10"
            :per-page="perPage" style="width=100; margin-right: auto"
            @paginate="myCallback"/>
        </div>

        </div>
        
    </div>
</template>

<script>
import axios from 'axios'
import Loading from '../components/Loading.vue'
import Pagination from 'vue-pagination-2';

export default {
    components: {
        Pagination,
        Loading
    },

    data: () => ({
        books: [],
        searchQuery: "",
        loading: false,
        page: 1,
        perPage: 4,
    }),
    mounted () {
        this.fetchBooks()
    },
    methods: {
        async fetchBooks () {
        this.loading = true
            try {
                const result = await axios.get(`https://www.anapioficeandfire.com/api/books?page=${this.page}&pageSize=${this.perPage}`)
                this.books = result.data
                this.loading = false
            } catch (error) {
                alert(error.message)
                this.loading = false
            }
        },

        renderList(pageNumber=1){
            //clear currently displayed list
            this.countriesToDisplay = [];

            //set countries to display
            if(this.allCountries.length){
                let _this = this;

                return new Promise(function(res){
                    //set the page to open to the pageNumber in the parameter in order to allow start and stop to update accordingly
                    _this.pageToOpen = pageNumber;

                    //add the necessary data to `countriesToDisplay` array
                    for(let i = _this.start; i <= _this.stop; i++){
                        _this.booksToDisplay.push(_this.allCountries[i]);
                    }

                    res();
                }).then(function(){
                    //Now update the current page to the page we just loaded
                    _this.currentPage = _this.pageToOpen;
                }).catch(function(){
                    console.log('render err');
                });                  
            }
        }

        
        
    },
    computed: {
        myCallback() {
            let pageNumber = this.page - 1;
            let paginatedBook = this.books.slice(
                pageNumber * this.perPage,
                (pageNumber + 1) * this.perPage
            )
            return paginatedBook;
        },


        
        filteredBooks() {
            const query = this.searchQuery.toLowerCase();
            if (this.searchQuery === "") {
                return this.books
            }

            return this.books.filter(book => {
                return Object.values(book).some(name => String(name).toLowerCase().includes(query))
            })
        },


        totalPages(){
            //calculate the total number of pages based on the number of items to show per page and the total items we got from server
            return this.allCountries.length && (this.allCountries.length > this.perPage) ? Math.ceil(this.allCountries.length/this.perPage) : 1;
        },

        start(){
            return (this.pageToOpen - 1) * this.perPage;
        },

        stop(){
            //stop at the end of the array if array length OR the items left are less than the number of items to show per page
            //do the calculation if otherwise
            if((this.allCountries.length - this.start) >= this.perPage){
                return (this.pageToOpen * this.perPage) - 1;
            }

            else{
                return this.allCountries.length - 1;
            }
        },
			
        showNext(){
            return this.currentPage < this.totalPages;
        },

        showPrev(){
            return this.currentPage > 1;
        }

    },
    watch: {
        //re-render list based on the value of `perPage` which indicates how many to show per page
        perPage: function(){
            this.renderList();
        }
    }
}
</script>



<style>
.home {
    max-width: 950px;
    width: 100%;
    margin: 1rem auto 0;
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
    padding: 2px 8px;
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
    white-space: nowrap !important;
    overflow: hidden;
    text-overflow: ellipsis !important;
}
td, th {
    border: 1px solid #dddddd !important;
    text-align: left;
    padding: 10px;
}

tr:nth-child(even) {
    background-color: #dddddd;
}

@media screen and (max-width:767px) {
    .home {
        width: 100%;
        margin: 0 auto;
    }
}
</style>
