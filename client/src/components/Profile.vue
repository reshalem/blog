<template>
    <div class="row" v-if="isLogin">
        <div class="col-lg-5">
            <div class="d-flex flex-column">
                <div class="card mb-3">
                    <div class="card-header">User Profile</div>
                    <div class="card-body">
                        <img class="card-img-top img-fluid mb-4" v-bind:src="user.avatarURL" alt="">
                        <p>Name: {{user.name}}</p>
                        <p class="mb-0">Email: {{user.email}}</p>
                    </div>
                </div>
                <div class="card mb-5">
                    <div class="card-header">Your Article List</div>
                    <div class="card-body">
                        <div class="d-flex flex-column justify-content-start">
                            <ul class="navbar-nav">
                                <li class="nav-item mb-3" v-for="article in articles">
                                    <router-link :to="{ name: 'id', params: { id: article._id }}">{{ article.title }}</router-link>
                                </li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="col-lg-7">
            <div class="d-flex flex-column">
                <div v-for="article in articles" class="card mb-4">
                    <div class="card-body">
                        <div v-if="isLogin && userEmail === article.author.email" class="d-flex justify-content-end align-items-center mb-2">
                            <i class="fas fa-fingerprint text-danger mr-2" id="btn-fingerprint"></i>
                            Your Article
                        </div>
                        <h4 class="card-title">{{ article.title }}</h4>
                        <p class="card-text">{{ article.description }}</p>
                        <div class="d-flex justify-content-between align-items-center mb-4">
                            <div class="font-weight-bold ml-5">By {{ article.author.name }}</div>
                            <i class="far fa-comment-alt mr-5" id="comment-symbol"></i>
                        </div>
                        <button class="btn btn-block font-weight-bold" id="btn-post" @click="goToLink(article._id)">
                            Read More
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import config from '@/config.js'

export default {
    name: 'Profile',
    props: ['isLogin', 'userEmail', 'checktoken'],
    data() {
        return {
            articles: [],
            user: {},
            shouldUpdate: 0
        }
    },
    methods: {
        getUserProfile: function() {
            axios({
                method: 'GET',
                url: `${config.port}/users`,
                headers: {
                    'access-token': localStorage.getItem('token')
                }
            })
                .then((user) => {
                    this.user = user.data
                })
                .catch((err) => {
                    console.log(err)
                })
        },
        goToLink: function(paramsId) {
            this.$router.push(`/articles/${paramsId}`)
        },
        getUserArticle: function() {
            axios({
                method: 'GET',
                url: `${config.port}/articles/ownsarticle/user`,
                headers: {
                    'access-token': localStorage.getItem('token')
                }
            })
                .then((articles) => {
                    this.articles = articles.data
                    console.log(articles)
                })
                .catch((err) => {
                    console.log(err)
                })
        }
    },
    created() {
        this.getUserArticle()
        this.getUserProfile()
    },
    mounted() {
        this.checktoken();
    }
}
</script>

<style>

</style>

