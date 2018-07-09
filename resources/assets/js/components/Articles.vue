<template>
	<div>
	<h2>Articles</h2>
	<form @submit.prevent="saveArticle" class="mb-3">
		<input type="text" 
			   v-model="article.title" class="form-control" placeholder="Title"></input>
		<textarea class="form-control" placeholder="Body"
				v-model="article.body"></textarea>

		<button type="submit" class="btn btn-light btn-block" v-bind:class="[ { 'disabled': article.title.length == 0 && article.body.length == 0 } ]">Save</button>
		
	</form>
	<nav aria-label="Page navigation example">
	  <ul class="pagination">
	    <li v-bind:class="[{ disabled: !pagination.prev_page_url }]"
	    	class="page-item">
	    	<a class="page-link" href="#"
	    	@click="fetchArticles(pagination.prev_page_url)">Previous</a></li>


	    <li v-for="n in pagination.last_page" class="page-item" v-bind:class="[ { disabled: n == pagination.current_page } ]">
	    	<a class="page-link" href="#" @click="fetchArticles( 'api/articles?page='+n )"> {{ n }} </a>
	    </li>

	    <li v-bind:class="[{ disabled: !pagination.next_page_url }]"
	    	class="page-item">
	    	<a class="page-link" href="#"
	    	@click="fetchArticles(pagination.next_page_url)">Next</a></li>
	  </ul>
	</nav>
	<div class="card card-body mb-2" v-for="article in articles"
	     v-bind:key="article.id">
	  <h3>{{ article.title }}</h3>
	  <p>{{ article.body }}</p>
	  <button v-on:click="editArticle(article)" class="btn btn-warning mb-3">Edit</button>
	  <button v-on:click="deleteArticle(article.id)" class="btn btn-danger">Delete</button>
	</div>
	</div>
</template>

<script>
	export default {
		data(){
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
		  }		  
		},
		created(){
			  this.fetchArticles();
		},
		methods: {
		  fetchArticles(page_url){

			let vm = this
			let page = page_url || '/api/articles'
			fetch(page)
			    .then(res => res.json())
			    .then(res => {
					this.articles = res.data
					vm.makePagination(res.meta, res.links)
			    })
		  },
		  makePagination(meta, links){
			let pagination = {
				current_page: meta.current_page,
				last_page: meta.last_page,
				next_page_url: links.next,
				prev_page_url: links.prev	
			}
			this.pagination = pagination
		  },

		  saveArticle(){
		  	let method = (this.edit ? 'put' : 'post')

		  	fetch('api/article', {
		  		method,
		  		body: JSON.stringify(this.article),
		  		headers: { 
		  			'Content-type': 'application/json'
		  		}
		  	}).then(res => res.json())
		  	  .then(data => {
		  	  	this.article.title = ''
		  	  	this.article.body = ''
		  	  })
		  	  this.fetchArticles()
		  	

		  },
		  editArticle(article){
		  	this.edit = true
		  	this.article.id = article.id
		  	this.article.article_id = article.id
		  	this.article.title = article.title
		  	this.article.body = article.body
		  },
		  deleteArticle(id){
		  	if(confirm('Are You sure you want to delete this article?'))
		  		fetch(`/api/article/${id}`, { 
		  			'method': 'delete'
		  		})
		  		.then(res => res.json())
		  		.then(data => {
		  			this.fetchArticles()
		  		})
		  }
			
		}
	}
</script>
