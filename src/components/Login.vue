<template>
  <div>
    <form v-if="!user" @submit.prevent="signUp">
      <input v-model="email" placeholder="Email" type="email" required />
      <input v-model="password" placeholder="Password" type="password" required />
      <button type="submit">Sign Up</button>
    </form>

    <form v-if="!user" @submit.prevent="signIn">
      <input v-model="email" placeholder="Email" type="email" required />
      <input v-model="password" placeholder="Password" type="password" required />
      <button type="submit">Sign In</button>
    </form>

    <button v-if="user" @click="signOut">Logout</button>
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
      const { error } = await supabase.auth.signInWithPassword({
        email: this.email,
        password: this.password
      })
      if (error) alert(error.message)
    },

  }
}
</script>