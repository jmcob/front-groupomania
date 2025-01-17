<template>
    <div class="post_comments_container">
        <div class="one-post">
            <div class="title_author_date">
                <h2 class="title">{{ post.title }}</h2>
                <div class="author_date">
                    <h3 class="user_id">
                        <img
                            class="avatar"
                            :src="this.poster.avatar"
                            alt="image de l'auteur du post"
                        />
                        <em>{{ this.poster.username }}</em
                        >,
                        {{ this.postInterval }}
                    </h3>
                </div>
                <hr />
            </div>
            <div class="text_buttons">
                <div v-if="hard">
                    <p class="text">{{ post.text }}</p>
                </div>
                <div v-else>
                    <textarea
                        rows="4"
                        cols="30"
                        v-model="updatedText"
                        placeholder="insérez un texte"
                    ></textarea>
                </div>
                <br />
                <div v-if="hard">
                    <div class="img" v-if="post.image">
                        <img :src="post.image" alt="image postée" />
                    </div>
                </div>
                <div v-else>
                    <input
                        class="file-input"
                        type="file"
                        accept="image/*"
                        name="image"
                        ref="image"
                        @change="handleFileUpload($event)"
                    />
                </div>
                <br />
                <div class="buttonContainer">
                    <div v-if="this.user">
                        <div
                            v-if="this.postLikedByCurrentUser"
                            @click="unlike"
                            :class="{
                                activeHeart: this.postLikedByCurrentUser,
                            }"
                        >
                            <p>
                                <i class="fas fa-heart"></i>
                                {{ this.count }}
                            </p>
                        </div>
                        <div v-else @click="like">
                            <p>
                                <i class="fas fa-heart"></i>
                                {{ this.count }}
                            </p>
                        </div>
                    </div>

                    <div v-if="this.match" class="buttons">
                        <div class="button" v-if="hard">
                            <p @click="hard = !hard">
                                <i class="far fa-edit"></i>Modifier
                            </p>
                        </div>
                        <div class="button" v-else>
                            <p
                                @click="
                                    $emit(
                                        'update-post',
                                        post.id,
                                        updatedText,
                                        form.image
                                    ),
                                        (hard = !hard)
                                "
                            >
                                <i class="far fa-paper-plane"></i>Envoyer
                            </p>
                        </div>
                        <div>
                            <p>
                                <i
                                    class="delete fas fa-times"
                                    @click="$emit('delete-post', post.id)"
                                >
                                </i
                                >Supprimer
                            </p>
                        </div>
                    </div>
                    <div v-else class="buttons"></div>
                </div>
            </div>
            <br />
        </div>
        <hr />
        <div class="comments">
            <div v-if="!this.user.logged"></div>
            <div v-else>
                <NewComment :post="post" @add-comment="addComment" />
            </div>

            <OneComment
                v-for="comment in comments"
                :comment="comment"
                :post="post"
                :key="comment.id"
            />
        </div>
    </div>
</template>

<script>
import OneComment from "./OneComment.vue";
import NewComment from "./NewComment.vue";
import { mapMutations, mapActions } from "vuex";

export default {
    components: { OneComment, NewComment },
    name: "OnePost",
    props: ["post", "updatePost"],
    data() {
        return {
            comments: [],
            hard: true, // if 'hard' property is  set to false, post is editable
            updatedText: "",
            poster: [],
            count: Number,
            postLikedByCurrentUser: false,
            match: Boolean,
            postInterval: "",
            form: { image: "" },
            user: {
                token: "",
                logged: false,
                user_id: Number,
                username: "",
                email: "",
                admin: Boolean,
            },
        };
    },

    methods: {
        ...mapMutations(["logOut"]),
        ...mapActions(["whoAmI"]),
        async addComment(newComment) {
            const comment = JSON.stringify(newComment);
            const res = await fetch("http://localhost:3000/api/comment/", {
                method: "POST",
                headers: {
                    "Content-type": "application/json",
                    Authorization: "Bearer " + this.user.token,
                },
                body: comment,
            }).then((res) => {
                return res.json();
            });
            const newBornComment = res.data;
            newBornComment.createdAt = "maintenant";
            this.comments = [newBornComment, ...this.comments];
        },
        handleFileUpload(e) {
            this.form.image = e.target.files[0];
        },
        async getComments() {
            const res = await fetch("http://localhost:3000/api/comment/").then(
                (res) => {
                    return res.json();
                }
            );
            return res.data;
        },
        idsMatch() {
            let posterId = this.post.user_id;
            let userid = this.user.user_id;
            let itsAMatch = posterId === userid || this.user.admin === true;
            return itsAMatch;
        },
        async whoIsTheAuthor() {
            let posterId = this.post.user_id;
            let res = await fetch(
                "http://localhost:3000/api/user/" + posterId
            ).then((res) => {
                return res.json();
            });
            this.poster.username = res.data.username;
            this.poster.email = res.data.email;
            this.poster.avatar = res.data.avatar;
        },
        async like() {
            if (this.user) {
                let json = {
                    post_id: this.post.id,
                    user_id: this.user.user_id,
                    admin: this.user.admin,
                };
                const res = await fetch("http://localhost:3000/api/like/", {
                    method: "POST",
                    headers: {
                        "content-type": "application/json",
                        Authorization: "Bearer " + this.user.token,
                    },
                    body: JSON.stringify(json),
                }).then((res) => {
                    return res.json();
                });
                if (res.data) {
                    this.count++;
                    this.postLikedByCurrentUser = true;
                }
            }
        },
        async unlike() {
            const json = JSON.stringify({
                post_id: this.post.id,
                user_id: this.user.user_id,
                admin: this.user.admin,
            });
            const res = await fetch("http://localhost:3000/api/like/", {
                method: "DELETE",
                headers: {
                    "content-type": "application/json",
                    Authorization: "Bearer " + this.user.token,
                },
                body: json,
            }).then((res) => {
                return res.json();
            });
            if (res.message) {
                this.count--;
                this.postLikedByCurrentUser = false;
            } else {
                alert("Erreur dans la suppression du like");
            }
        },
        async counter() {
            const res = await fetch(
                "http://localhost:3000/api/like/" + this.post.id
            ).then((res) => {
                return res.json();
            });
            if (res.data.length != null) return res.data.length;
        },
        async getOneLike() {
            const json = JSON.stringify({
                post_id: this.post.id,
                user_id: this.user.user_id,
            });
            const res = await fetch(
                "http://localhost:3000/api/like/getASpecificLike",
                {
                    method: "POST",
                    headers: {
                        "content-type": "application/json",
                        Authorization: "Bearer " + this.user.token,
                    },
                    body: json,
                }
            ).then((res) => {
                return res.json();
            });
            if (res.data) {
                let liked = true;
                return liked;
            }
        },
        timestamp() {
            let timestamp;
            if (this.post.createdAt == "maintenant") {
                timestamp = "maintenant";
            } else {
                let t = this.post.createdAt.split(/[- :]/);
                let d = new Date(
                    Date.UTC(t[0], t[1] - 1, t[2], t[3], t[4], t[5])
                );
                timestamp = d.toLocaleString();
            }
            return timestamp;
        },
    },
    async created() {
        this.comments = await this.getComments();
        this.count = await this.counter();
        this.user = await this.whoAmI();
        this.match = this.idsMatch();
        await this.whoIsTheAuthor();
        this.postLikedByCurrentUser = await this.getOneLike();
        this.postInterval = this.timestamp();
    },
};
</script>

<style scoped>
.activeHeart {
    color: crimson;
}
.post_comments_container {
    border: 1px solid grey;
    border-radius: 5px;
    width: 80vw;
    margin: 0 auto;
}
.one-post {
    display: flex;
    flex-direction: column;
    margin: 0px;
}
.title_author_date {
    display: flex;
    flex-direction: column;
    justify-content: center;
}
.title {
    margin: 0px;
}
.img img {
    width: 300px;
}
.avatar {
    width: 35px;
    height: 35px;
}
.author_date {
    display: flex;
    justify-content: space-around;
    align-items: center;
}
.text_buttons {
    display: flex;
    flex-direction: column;
    justify-content: space-around;
    align-items: center;
}
.buttons {
    display: flex;
    flex-direction: column;
    padding: 0px;
}
.buttons,
p {
    margin: 0px;
}
.comments {
    display: flex;
    flex-direction: column;
}
.title {
    background-color: rgb(255, 221, 227);
}
.fa-heart:hover,
.fa-edit:hover,
.fa-paper-plane:hover,
.fa-times:hover {
    color: pink;
}
.fa-heart:active,
.fa-edit:active,
.fa-paper-plane:active,
.fa-times:active {
    color: crimson;
}
i {
    font-size: x-large;
    padding: 8px;
}
</style>
