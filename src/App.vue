<template>
    <Login v-if="!user"/>
    <el-container v-else>
      <el-header>
        <div class="logout_con">
         <el-dropdown>
          <span class="el-dropdown-link">
            Hi! {{ name }} <i class="fa-solid fa-angle-down"></i>
          </span>
          <template #dropdown>
            <el-dropdown-menu>
              <el-dropdown-item @click="signOut">Logout</el-dropdown-item>
            </el-dropdown-menu>
          </template>
        </el-dropdown>
        </div>
      </el-header>
      <el-main>
        
        <RouterView />
      </el-main>
    </el-container>
  
</template>

<script>
import { supabase } from './lib/supabaseClient'
import Login from './components/Login.vue';
export default{
  components: {Login},
  data(){
    return {
      name: 'User',
      user: ''
    }
  },
  methods: {
    async signOut() {
      await supabase.auth.signOut()
    }
  },
  async mounted(){
    const { data } = await supabase.auth.getSession()
    this.user = data.session?.user || null

    supabase.auth.onAuthStateChange((_, session) => {
      this.user = session?.user || null
    })
  }
}
</script>
