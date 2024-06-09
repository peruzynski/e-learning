<template>
  <div class="top-bar">
    <div class="site-name">{{ courseName }}</div>
    <ThemeSwitcher @theme-change="changeTheme" class="theme-switcher" />
  </div>
</template>

<script>
import ThemeSwitcher from "./ThemeSwitcher.vue";

export default {
  components: {
    ThemeSwitcher,
  },
  props: {
    courseName: {
      type: String,
      default: "Course Name",
    },
  },
  methods: {
    changeTheme(newTheme) {
      this.$emit("theme-change", newTheme);
    },
  },
  mounted() {
    console.log("Course Name Prop:", this.courseName); // Debugging line
  },
};
</script>

<style scoped>
.top-bar {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 10px 20px;
  background-color: var(--nav-bg-color);
  color: var(--nav-text-color);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  position: relative;
}

.site-name {
  font-size: 1.5em;
  font-weight: bold;
}

.theme-switcher {
  position: absolute;
  right: 20px;
}
</style>
