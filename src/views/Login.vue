<template>
    <form @submit.prevent="login">
        <h2>{{ title }}</h2>
        <CustomLoginInput class="input" v-model="email" :label="emailLabel" />
        <CustomLoginInput
            class="input"
            v-model="password"
            :label="passwordLabel"
        />
        <button @click="$emit('submitted', data.user_id)">
            {{ title }}
        </button>
    </form>
    <br />
    <router-link to="/">Page d'accueil</router-link>
</template>

<script>
import CustomLoginInput from "../components/CustomLoginInput";

export default {
    name: "Login",
    components: { CustomLoginInput },
    data() {
        return {
            title: "Connexion",
            emailLabel: "Email :",
            passwordLabel: "Mot de passe :",
            data: Object,
            email: "",
            password: "",
        };
    },
    methods: {
        async login() {
            let user = {
                email: this.email,
                password: this.password,
            };
            let userJSON = JSON.stringify(user);
            const userData = await fetch(
                "http://localhost:3000/api/user/login",
                {
                    method: "POST",
                    headers: {
                        "Content-type": "application/json",
                    },
                    body: userJSON,
                }
            ).then((res) => {
                return res.json();
            });
            if (userData.user_id) {
                localStorage.setItem("userData", JSON.stringify(userData));
                this.$router.push("/");
            }
        },
    },
};
</script>

<style scoped>
.input {
    margin: 10px;
    display: block;
}
</style>
