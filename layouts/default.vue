<template>
  <div class="familytree">
    <header-layout />

    <main class="main">
      <!-- Nội dung của trang sẽ được hiển thị ở đây -->
      <nuxt />
    </main>

    <footer-layout v-if="showFooter" />
  </div>
</template>

<script>
import footerLayout from '../components/footerLayout.vue'
import headerLayout from '../components/headerLayout.vue'
export default {
  components: { headerLayout, footerLayout },
  computed: {
    showFooter() {
      // Điều kiện để hiển thị footerLayoutå
      return !(
        this.$route.path === '/account/dang_nhap' ||
        this.$route.path === '/account/dang_ki' ||
        this.$route.path === '/account/quen_mat_khau' ||
        this.$route.path === '/account/xac_nhan_otp' ||
        this.$route.path === '/lich_su_chinh_sua'
      )
    },
  },
}
</script>

<style scoped>
main {
  background-color: #fff1dc;
}

</style>
