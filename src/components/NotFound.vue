<template>
<div class="not_found_con">
  <el-empty
    :image-size="200"
    description="Looks like the page your are looking for does not exist"
  />
  <el-button @click="gotoIndex" type="primary">Go to Home</el-button>
</div>
</template>

<script lang="ts">
export default {
  data() {
    return {}
  },

  methods: {
    gotoIndex() {
      this.$router.push('/')
    },
  },
}
</script>

<style scoped></style>