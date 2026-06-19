<template>
    <el-container>
      <el-header>
        <div class="logout_con">
         <el-dropdown>
          <span class="el-dropdown-link">
            Hi! {{ name }} <i class="fa-solid fa-angle-down"></i>
          </span>
          <template #dropdown>
            <el-dropdown-menu>
              <el-dropdown-item>Action 1</el-dropdown-item>
              <el-dropdown-item>Action 2</el-dropdown-item>
              <el-dropdown-item>Action 3</el-dropdown-item>
              <el-dropdown-item disabled>Action 4</el-dropdown-item>
              <el-dropdown-item divided>Action 5</el-dropdown-item>
            </el-dropdown-menu>
          </template>
        </el-dropdown>
        </div>
      </el-header>
      <el-main><RouterView /></el-main>
    </el-container>
  
</template>

<script>
export default{
  data(){
    return {
      name: 'User'
    }
  }
}
</script>
