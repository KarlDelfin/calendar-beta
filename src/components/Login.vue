<template>
  <div>
    <form v-if="!user">
      <el-input v-model="email" placeholder="Email" type="email" required />
      <el-input v-model="password" placeholder="Password" type="password" required />
      <el-button type="submit" @click="signUp">Sign Up</el-button>
    </form>

    <el-form v-if="!user" @submit.prevent="signIn">
      <el-input v-model="email" placeholder="Email" type="email" required />
      <el-input v-model="password" placeholder="Password" type="password" required />
      <el-button type="submit" @click="signIn">Sign In</el-button>
    </el-form>

    <el-button v-if="user" @click="signOut">Logout</el-button>
    <p v-if="user">Welcome, {{ user.email }}</p>
  </div>
</template>

<script>
import { supabase } from '../lib/supabaseClient'

export default {
  data() {
    return {
      email: '',
      password: '',
      user: null
    }
  },
  async mounted() {
    const { data } = await supabase.auth.getSession()
    this.user = data.session?.user || null

    supabase.auth.onAuthStateChange((_, session) => {
      this.user = session?.user || null
    })
  },
  methods: {
    async signUp() {
      const { error } = await supabase.auth.signUp({
        email: this.email,
        password: this.password
      })
      if (error) alert(error.message)
    },
    async signIn() {
      const { data, error } = await supabase.auth.signInWithPassword({
        email: this.email,
        password: this.password
      })
      if (error) alert(error.message)

      this.$store.dispatch('setUser', data.user)
      console.log('User ID:', this.$store.getters.getUser)
    },
  }
}
</script>