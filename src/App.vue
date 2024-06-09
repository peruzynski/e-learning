<template>
  <div :class="theme">
    <Board
      course-learn-endpoint="http://porzuczek.pl/learn/22S22ISI32/s213shabibiwclg1"
      @theme-change="changeTheme"
    />
  </div>
</template>

<script>
import Board from "./components/Board.vue";

export default {
  name: "App",
  components: {
    Board,
  },
  data() {
    return {
      theme: "light",
    };
  },
  methods: {
    changeTheme(newTheme) {
      this.theme = newTheme;
    },
  },
  created() {
    const savedTheme = localStorage.getItem("theme") || "light";
    this.theme = savedTheme;
    document.documentElement.setAttribute("data-theme", savedTheme);
  },
};
</script>

<style>
:root {
  --bg-color: #fff;
  --text-color: #000;
  --highlight-color: #007bff;
  --nav-bg-color: #f8f9fa;
  --nav-text-color: #000;
  --nav-hover-bg-color: #e9ecef;
}

[data-theme="dark"] {
  --bg-color: #121212;
  --text-color: #e0e0e0;
  --highlight-color: #bb86fc;
  --nav-bg-color: #333;
  --nav-text-color: #e0e0e0;
  --nav-hover-bg-color: #444;
}

body {
  background-color: var(--bg-color);
  color: var(--text-color);
}

.active {
  background-color: var(--highlight-color);
  color: #fff;
}
</style>
