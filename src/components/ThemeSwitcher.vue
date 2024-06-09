<template>
  <div class="theme-switcher">
    <label class="switch">
      <input type="checkbox" v-model="isDarkTheme" @change="toggleTheme" />
      <span class="slider round"></span>
    </label>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isDarkTheme: false,
    };
  },
  methods: {
    toggleTheme() {
      this.$emit('theme-change', this.isDarkTheme ? 'dark' : 'light');
      document.documentElement.setAttribute('data-theme', this.isDarkTheme ? 'dark' : 'light');
    },
  },
  created() {
    this.isDarkTheme = localStorage.getItem('theme') === 'dark';
    document.documentElement.setAttribute('data-theme', this.isDarkTheme ? 'dark' : 'light');
  },
  watch: {
    isDarkTheme(newVal) {
      localStorage.setItem('theme', newVal ? 'dark' : 'light');
    },
  },
};
</script>

<style scoped>
.theme-switcher {
  position: fixed;
  top: 10px;
  right: 10px;
  z-index: 1001;
}

.switch {
  position: relative;
  display: inline-block;
  width: 60px;
  height: 34px;
}

.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  transition: .4s;
  border-radius: 34px;
}

.slider:before {
  position: absolute;
  content: "";
  height: 26px;
  width: 26px;
  left: 4px;
  bottom: 4px;
  background-color: white;
  transition: .4s;
  border-radius: 50%;
}

input:checked + .slider {
  background-color: #2196F3;
}

input:checked + .slider:before {
  transform: translateX(26px);
}
</style>
