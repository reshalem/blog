<template>
    <div>
        <div class="d-flex justify-content-start mb-2">
            <button class="btn btn-link" @click="backToAll"><i class="fas fa-undo-alt"></i> Back</button>
        </div>
        <div class="card mb-3">
            <div class="card-body">
                <div v-if="isLogin && userEmail === article.author.email" class="d-flex justify-content-end align-items-center mb-4">
                    <i class="fas fa-edit text-dark mr-3" data-toggle="modal" data-target="#editModal" id="btn-edit-delete"></i>
                    <i class="fas fa-minus-circle text-danger" id="btn-edit-delete" data-toggle="modal" data-target="#confirmModalDelete"></i>
                </div>
                <EditModal :articleId="article._id" :getonearticle="getOneArticle"></EditModal>
                <div class="modal fade mt-5" id="confirmModalDelete">
                    <div class="modal-dialog">
                        <div class="modal-content">
                            <div class="modal-body">
                                <div class="mb-4 mt-3">Are you sure you want to delete this article?</div>
                                <div class="mb-3">
                                    <button class="btn btn-danger font-weight-bold" data-dismiss="modal">Cancel</button>
                                    <button class="btn btn-primary font-weight-bold ml-5" data-dismiss="modal" @click="deleteArticle(article._id)">Positive</button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <h4 class="card-title">{{ article.title }}</h4>
                <div class="container mb-4">
                    <p class="card-text text-justify">{{ article.description }}</p>
                </div>
                <div class="d-flex justify-content-between align-items-center mb-2">
                    <div class="font-weight-bold ml-5" v-if="article.author">By {{ article.author.name }}</div>
                    <div class="d-flex justify-content-between align-items-center mb-2">
                        <i class="far fa-comment-alt mr-2" id="comment-symbol"></i>
                        <div class="lead font-weight-bold mr-5">{{commentBadge}}</div>
                    </div>
                </div>
                <div class="text-right mr-5" @click="showCommentForm" style="cursor:pointer">See comments</div>
            </div>
        </div>
        <div class="card mb-5" v-if="commentForm">
            <div class="h3 text-left ml-4 mt-4 mb-3" id="comment-section-h">Comment Section</div>
            <div class="card-body">
                <textarea type="text" class="form-control mb-3" placeholder="Add Comment" v-model="commentContent"></textarea>
                <div class="d-flex justify-content-end mb-4">
                    <input @click="addComment" type="submit" class="btn btn-block btn-primary font-weight-bold" value="Submit">
                </div>
                <div class="d-flex flex-column">
                    <div v-for="comment in comments" class="d-flex justify-content-between container mb-3 border border-bottom py-3">
                        <div>From: {{comment.user.name}}</div>
                        <div class="d-flex flex-column justify-content-end">
                            <div class="mb-2">{{comment.content}}</div>
                            <i class="far fa-trash-alt text-warning text-right" id="btn-edit-delete" v-if="userEmail === comment.user.email" @click="deleteComment(comment._id)"></i>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import EditModal from '@/components/EditModal.vue'
import config from '@/config.js'

export default {
    name: 'OneArticle',
    props: ['id', 'isLogin', 'userEmail', 'getarticles', 'checktoken'],
    components: {
        EditModal
    },
    data() {
        return {
            article: {},
            articleId: '',
            comments: '',
            commentContent: '',
            commentBadge: 0,
            commentForm: false
        }
    },
    methods: {
        getOneArticle: function() {
            axios({
                method: 'GET',
                url: `${config.port}/articles/${this.id}`
            })
                .then((article) => {
                    this.article = article.data
                })
                .catch((err) => {
                    console.log(err)
                });
        },
        getComments: function() {
            axios({
                method: 'GET',
                url: `${config.port}/comments/${this.id}`,
                headers: {
                    'access-token': localStorage.getItem('token')
                }
            })
                .then((comments) => {
                    this.comments = comments.data
                    console.log(comments)
                    this.commentBadge = this.comments.length
                })
                .catch((err) => {
                    console.log(err)
                })
        },
        addComment: function() {
            axios({
                method: 'POST',
                url: `${config.port}/comments/${this.id}`,
                data: {
                    content: this.commentContent
                },
                headers: {
                    'access-token': localStorage.getItem('token')
                }
            })
                .then((result) => {
                    this.commentContent = ''
                    this.getComments()
                })
                .catch((err) => {
                    console.log(err)
                })
        },
        deleteComment: function(commentId) {
            axios({
                method: 'DELETE',
                url: `${config.port}/comments/${this.id}`,
                headers: {
                    'access-token': localStorage.getItem('token')
                },
                data: {
                    id: commentId
                }
            })
                .then((result) => {
                    this.getComments();
                })
                .catch((err) => {
                    console.log(err)
                })
        },
        showCommentForm: function() {
            if (this.commentForm === true) {
                this.commentForm = false
            } else {
                this.commentForm = true
            }
        },
        backToAll: function() {
            this.$router.push('/articles')
        },
        deleteArticle: function(articleId) {
            axios({
                method: 'DELETE',
                url: `${config.port}/articles/${articleId}`,
                headers: {
                    'access-token': localStorage.getItem('token')
                }
            })
                .then((result) => {
                    this.getarticles()
                    this.backToAll()
                })
                .catch((err) => {
                    console.log(err)
                })
        }
    },
    created() {
        this.getOneArticle()
        this.getComments()
        this.checktoken()
    },
    watch: {
        id(val) {
            this.articleId = val

            axios({
                method: 'GET',
                url: `${config.port}/articles/${this.articleId}`
            })
                .then((article) => {
                    this.article = article.data
                })
                .catch((err) => {
                    console.log(err)
                })
            this.getComments()
        }
    }
}
</script>

<style>
#comment-section-h {
    font-family: 'Oswald', serif;
}
</style>
