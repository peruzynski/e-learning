<template>
  <div class="board-container">
    <div class="top-bar-container">
      <TopBar
        :course-name="currentCourse.title"
        @theme-change="handleThemeChange"
        @toggle-navigation="toggleNavigation"
      />
    </div>
    <div class="main-content">
      <Navigation
        :course="[currentCourse]"
        :highlight-color="highlightColor"
        :selected-language="selectedLanguage"
        :api-key="apiKey"
        :is-nav-visible="isNavVisible"
        @select-content="changeCurrentContent"
      />
      <div class="content-panel">
        <div class="language-selector">
          <select v-model="selectedLanguage" @change="handleLanguageChange">
            <option value="en">English</option>
            <option value="pl">Polski</option>
            <option value="es">Español</option>
            <!-- Add other languages as needed -->
          </select>
        </div>
        <div
          v-html="translatedContent || defaultContent"
          class="translated-content"
        ></div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Navigation from "./Navigation.vue";
import TopBar from "./TopBar.vue";

export default {
  name: "Board",
  components: {
    Navigation,
    TopBar,
  },
  data() {
    return {
      courses: [],
      currentCourse: {},
      currentCourseTopicId: null,
      currentCourseQuestId: null,
      highlightColor: getComputedStyle(document.documentElement)
        .getPropertyValue("--highlight-color")
        .trim(),
      translatedContent: "",
      selectedLanguage: "pl", // Default language set to Polish
      apiKey: "AIzaSyBJKjlYsF-h1YlI7CCstcP1dCMvoq11p3U", // Replace with your API key
      isNavVisible: false,
      defaultContent: "<p>Loading content...</p>", // Placeholder content
    };
  },
  methods: {
    async handleLanguageChange() {
      this.translateContent(); // Fetch the new content based on the selected language
    },
    async translateContent() {
      if (this.selectedLanguage === "pl") {
        this.translatedContent = this.current.content;
      } else {
        const target = this.selectedLanguage;
        const text = this.current.content;

        if (!text) return;

        const url = `https://translation.googleapis.com/language/translate/v2?key=${this.apiKey}`;
        const data = {
          q: text,
          target: target,
        };

        try {
          const response = await axios.post(url, data);
          this.translatedContent =
            response.data.data.translations[0].translatedText;
        } catch (error) {
          console.error("Translation error:", error);
          this.translatedContent = "<p>Error loading content...</p>";
        }
      }
    },
    changeCurrentContent(item) {
      if (item.content) {
        if (this.current.content !== item.content) {
          this.$root.$emit("reset-view");
          window.scrollTo(0, 0);
        }
        if (item.child) {
          this.currentCourseTopicId = item.id;
          this.currentCourseQuestId = null;
        } else {
          this.currentCourseTopicId = item.topic_id;
          this.currentCourseQuestId = item.id;
        }
        this.translateContent(); // Translate when content changes
      }
    },
    async fetch() {
      try {
        const coursesResponse = this.courseLearnEndpoint
          ? await axios.get(this.endpoint)
          : {};
        if (
          Array.isArray(coursesResponse.data) &&
          coursesResponse.data &&
          coursesResponse.data.length > 0
        ) {
          const coursesFormatted = this.format(coursesResponse.data);
          if (
            JSON.stringify(this.courses) !== JSON.stringify(coursesFormatted)
          ) {
            this.courses = coursesFormatted;
            this.currentCourse = this.courses[0];
            this.currentCourseTopicId = !this.currentCourseTopicId
              ? this.courses[0].topics[0].id
              : this.currentCourseTopicId;
            this.translateContent(); // Ensure translation is done after fetching courses
          }
        }
      } catch (ex) {
        this.courses = [];
      }
    },
    format(response) {
      return JSON.parse(
        JSON.stringify(response).replace(/"quests":/g, '"child":')
      );
    },
    handleThemeChange(newTheme) {
      this.$emit("theme-change", newTheme);
    },
    toggleNavigation() {
      this.isNavVisible = !this.isNavVisible;
    },
  },
  props: {
    refreshRate: { type: Number, default: 35000 },
    courseLearnEndpoint: { type: String, default: "" },
  },
  async created() {
    await this.fetch();
  },
  mounted() {
    this.translateContent(); // Translate content to the default language on mount
  },
  computed: {
    endpoint() {
      return `${this.courseLearnEndpoint}`;
    },
    current() {
      const current =
        this.currentCourse && this.currentCourseTopicId
          ? !this.currentCourseQuestId
            ? this.courses[0].topics.find(
                (item) => item.id === this.currentCourseTopicId
              )
            : this.courses[0].topics
                .find((item) => item.id === this.currentCourseTopicId)
                .child.find((item) => item.id === this.currentCourseQuestId)
          : {};
      const result = current
        ? current
        : (() => {
            this.currentCourseTopicId = this.courses[0].topics[0].id;
            this.currentCourseQuestId = null;
            return this.courses[0].topics.find(
              (item) => item.id === this.currentCourseTopicId
            );
          })();
      return result;
    },
  },
  watch: {
    selectedLanguage(newLang) {
      this.translateContent();
    },
    current() {
      this.translateContent();
    },
  },
};
</script>

<style scoped>
.board-container {
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.top-bar-container {
  flex: 0 0 auto;
}

.main-content {
  display: flex;
  flex: 1 1 auto;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

.content-panel {
  flex: 1;
  max-width: 800px;
  padding: 20px;
  background-color: var(--bg-color);
  color: var(--text-color);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  overflow-y: auto;
}

.language-selector {
  margin-bottom: 20px;
}

.translated-content {
  width: 100%;
}
</style>
