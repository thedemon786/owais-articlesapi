<template>
    <div>
        <h2>Articles</h2>
        <form @submit.prevent="addArticle" class="mb-3">
            <div class="form-group">
                <input type="text" class="form-control" ref="search" placeholder="Title" v-model="article.title">
            </div>
            <div class="form-group">
                <textarea class="form-control" placeholder="Body" v-model="article.body"></textarea>
            </div>
            <button type="submit" class="btn btn-success btn-block">Save</button>
        </form>
        <button @click="clearForm()" class="btn btn-info btn-block mb-2">Cancel</button>
        <nav aria-label="Page navigation example">
            <ul class="pagination">
                <li v-bind:class="[{disabled: !pagination.prev_page_url}]" class="page-item"><a class="page-link" href="#"
                        @click="fetchArticles(pagination.prev_page_url)">Previous</a></li>

                <li class="page-item disabled"><a class="page-link text-dark" href="#">Page {{ pagination.current_page
                        }} of {{ pagination.last_page }}</a></li>

                <li v-bind:class="[{disabled: !pagination.next_page_url}]" class="page-item"><a class="page-link" href="#"
                        @click="fetchArticles(pagination.next_page_url)">Next</a></li>
            </ul>
        </nav>
        <div class="container">
            <div class="row">
                <div class="card card-body mb-2 col-md-12" v-for="article in articles" v-bind:key="article.id">

                    <h3>{{ article.title }}</h3>
                    <p>{{ article.body }}</p>

                    <hr>
                    <button @click="editArticle(article); setFocus();" class="btn btn-warning mb-2">Edit</button>
                    <button @click="deleteArticle(article.id)" class="btn btn-danger">Delete</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                articles: [],
                article: {
                    id: '',
                    title: '',
                    body: ''
                },
                article_id: '',
                pagination: {},
                edit: false
            };
        },
        created() {
            this.fetchArticles();
        },
        methods: {
            fetchArticles(page_url) {
                let vm = this;
                page_url = page_url || '/api/articles';
                fetch(page_url)
                    .then(res => res.json())
                    .then(res => {
                        this.articles = res.data;
                        vm.makePagination(res.meta, res.links);
                    })
                    .catch(err => console.log(err));
            },
            makePagination(meta, links) {
                let pagination = {
                    current_page: meta.current_page,
                    last_page: meta.last_page,
                    curr_page_url: links.first.replace("1",meta.current_page),
                    next_page_url: (links.next == null) ? null : links.next.replace("http://","https://"),
                    prev_page_url: (links.prev == null) ? null : links.prev.replace("http://","https://")
                };
                this.pagination = pagination;
            },
            deleteArticle(id) {
                if (confirm('Are You Sure?')) {
                    fetch(`api/article/${id}`, {
                            method: 'delete'
                        })
                        .then(res => res.json())
                        .then(data => {
                            alert('Article Removed');
                            this.fetchArticles();
                        })
                        .catch(err => console.log(err));
                }
            },
            addArticle() {
                if (this.edit === false) {
                    // Add
                    fetch('api/article', {
                            method: 'post',
                            body: JSON.stringify(this.article),
                            headers: {
                                'content-type': 'application/json'
                            }
                        })
                        .then(res => res.json())
                        .then(data => {
                            this.clearForm();
                            alert('Article Added');
                            this.fetchArticles();
                        })
                        .catch(err => console.log(err));
                } else {
                    // Update
                    fetch('api/article', {
                            method: 'put',
                            body: JSON.stringify(this.article),
                            headers: {
                                'content-type': 'application/json'
                            }
                        })
                        .then(res => res.json())
                        .then(data => {
                            this.clearForm();
                            this.edit = false;
                            let cur_page_url= this.pagination.curr_page_url.replace("http://","https://");
                            alert('Article Updated');
                            // let cur_url = res.links.first.replace("1","res.meta.current_page");
                            this.fetchArticles(cur_page_url);
                        })
                        .catch(err => console.log(err));
                }
            },
            editArticle(article) {
                this.edit = true;
                this.article.id = article.id;
                this.article.article_id = article.id;
                this.article.title = article.title;
                this.article.body = article.body;
            },
            clearForm() {
                this.edit = false;
                this.article.id = null;
                this.article.article_id = null;
                this.article.title = '';
                this.article.body = '';
            },
            setFocus: function () {
                // Note, you need to add a ref="search" attribute to your input.
                this.$refs.search.focus();
            }
        }
    };

</script>
