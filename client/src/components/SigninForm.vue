<template>
    <div class="sign_page">
        <div v-show="signin_field">
            <div class="container_sign">
                <div class="signin">
                    <h4>Sign in Orion Kanban</h4>
                    <form @submit.prevent="signin">
                        <input 
                            v-model="signin_email" 
                            class="signin_username" 
                            type="email" 
                            placeholder="Enter email" required>
                        <input 
                            v-model="signin_password" 
                            class="signin_password" 
                            type="password" 
                            placeholder="Enter password" required>
                        <button type="submit" class="signin_button">Sign In</button>
                    </form>
                    <p>OR</p>
                    <button @click.prevent="signinGoogle" class="signin_google_button"><i class="fa fa-google"></i>Sign in with Google</button>
                    <a @click.prevent="toSignup" class="signin_register" href="#">Sign Up for an account</a>
                </div>
            </div>
        </div>

        <div v-show="signup_field">
            <div class="container_sign">
                <div class="signup">
                    <h4>Sign up for your account</h4>
                    <form @submit.prevent="signup">
                        <input 
                            v-model="signup_name" 
                            class="signup_name" 
                            type="text" 
                            placeholder="Enter your name" required>
                        <input 
                            v-model="signup_email" 
                            class="signup_email" 
                            type="email" 
                            placeholder="Enter your email" required>
                        <input 
                            v-model="signup_password" 
                            class="signup_password" 
                            type="password" 
                            placeholder="Enter your password" required>
                        <button class="signup_button">Sign Up</button>
                    </form>
                    <a href="#" class="cancel_signup_button" @click.prevent="toSignin">Cancel</a>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'

export default {
    name: 'SigninForm',
    props: ['url'],
    data(){
        return {
            signin_field: true,
            signup_field: false,
            signin_email: null, signin_password: null,
            signup_name: null, signup_email: null, signup_password: null,
        }
    },
    created(){
        
    },
    methods: {
        toSignup(){
            this.signin_field = false
            this.signup_field = true
        },

        toSignin(){
            this.signin_field = true
            this.signup_field = false
        },

        signup(){           
            axios({
                url: `${this.url}/user/signup`,
                method: 'POST',
                data: {
                    name: this.signup_name,
                    email: this.signup_email,
                    password: this.signup_password
                }
            })
                .then(() => {
                    this.signup_field = false
                    this.signin_field = true
                    this.signup_name = null
                    this.signup_email = null
                    this.signup_password = null
                    
                    const Toast = Swal.mixin({
                        toast: true,
                        position: 'top',
                        showConfirmButton: false,
                        timer: 3000,
                        timerProgressBar: true,
                        onOpen: (toast) => {
                            toast.addEventListener('mouseenter', Swal.stopTimer)
                            toast.addEventListener('mouseleave', Swal.resumeTimer)
                        }
                        })
                        Toast.fire({
                        icon: 'success',
                        position: 'top',
                        title: 'Signed Up Successfully'
                        })
                })
                .catch(err => {
                    console.log(err.response);
                })
        },
        
        signin(){
            axios({
                url: `${this.url}/user/signin`,
                method: 'POST',
                data: {
                    email: this.signin_email,
                    password: this.signin_password
                }
            })
                .then(userLogin => {
                    this.signin_email = null
                    this.signin_password = null

                    localStorage.setItem('token', userLogin.data.payload.token)
                    localStorage.setItem('name', userLogin.data.payload.user.name)
                    localStorage.setItem('email', userLogin.data.payload.user.email)
                    // this.$emit('toMain', false, true)
                    this.$emit('toMain')

                    const Toast = Swal.mixin({
                    toast: true,
                    position: 'top',
                    showConfirmButton: false,
                    timer: 3000,
                    timerProgressBar: true,
                    onOpen: (toast) => {
                        toast.addEventListener('mouseenter', Swal.stopTimer)
                        toast.addEventListener('mouseleave', Swal.resumeTimer)
                    }
                    })
                    Toast.fire({
                    icon: 'success',
                    title: 'Signed In Successfully'
                    })
                })
                .catch(err => {
                    console.log(err);

                    const Toast = Swal.mixin({
                        toast: true,
                        position: 'top',
                        showConfirmButton: false,
                        timer: 3000,
                        timerProgressBar: true,
                        onOpen: (toast) => {
                            toast.addEventListener('mouseenter', Swal.stopTimer)
                            toast.addEventListener('mouseleave', Swal.resumeTimer)
                        }
                    })
                    Toast.fire({
                        icon: 'error',
                        title: 'Invalid Email / Password'
                    })
                })
            
        },

        signinGoogle(){  
            console.log('masuk methods');
            this.$gAuth.signIn()
                .then(GoogleUser => {
                    let id_token = GoogleUser.getAuthResponse().id_token
                    console.log('masuk');
                    
                    axios({
                        url: `${url}/user/signin-google`,
                        method: 'POST',
                        headers: {
                            token: id_token
                        }
                    })
                        .then(gUser => {
                            localStorage.setItem('token', gUser.data.payload.token)
                            localStorage.setItem('name', gUser.data.payload.user.name)
                            localStorage.setItem('email', gUser.data.payload.user.email)
                        })
                        .catch(err => {
                            console.log(err);
                        })
                })
                .catch(err => {
                    console.log(err);
                    
                })
        },
    }
}
</script>