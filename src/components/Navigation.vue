<template>
  <div class="navigation-container" @mouseover="showNav" @mouseleave="hideNav">
    <div class="navigation" :class="{ 'navigation-visible': isNavVisible }">
      <div class="navigation-scroll">
        <div
          v-for="(topic, index) in translatedTopics"
          :key="index"
          class="menu-item"
        >
          <div
            class="menu-title"
            @click="toggleSubmenu(index)"
            :class="{ active: isTopicActive(topic) }"
            :style="{
              backgroundColor: isTopicActive(topic)
                ? highlightColor
                : 'transparent',
              color: isTopicActive(topic) ? '#fff' : 'inherit',
              paddingLeft: topic.child ? '10px' : '0',
            }"
          >
            {{ topic.title }}
          </div>
          <div v-show="isSubmenuOpen(index)" class="submenu">
            <div
              v-for="(child, childIndex) in topic.child"
              :key="childIndex"
              class="submenu-item"
            >
              <div
                class="submenu-title"
                @click="onItemClick(child)"
                :class="{ active: isChildActive(child) }"
                :style="{
                  backgroundColor: isChildActive(child)
                    ? highlightColor
                    : 'transparent',
                  color: isChildActive(child) ? '#fff' : 'inherit',
                }"
              >
                {{ child.title }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "Navigation",
  props: {
    course: { type: Array, default: () => [{}] },
    highlightColor: {
      type: String,
      default: getComputedStyle(document.documentElement)
        .getPropertyValue("--highlight-color")
        .trim(),
    },
    selectedLanguage: { type: String, default: "en" },
    apiKey: { type: String, required: true },
  },
  data() {
    return {
      isNavVisible: false,
      openSubmenus: [],
      translatedTopics: [],
    };
  },
  computed: {
    topics() {
      return this.course.length > 0 && this.course[0].topics
        ? this.course[0].topics
        : [];
    },
  },
  watch: {
    topics: {
      handler(newTopics) {
        this.translateTopics(newTopics);
      },
      immediate: true,
    },
    selectedLanguage(newLang) {
      this.translateTopics(this.topics);
    },
  },
  methods: {
    async translateTopics(topics) {
      if (!topics || topics.length === 0) return;

      if (this.selectedLanguage === "pl") {
        try {
          const response = await axios.get("https://api.porzuczek.pl/topics");
          this.translatedTopics = response.data.topics;
        } catch (error) {
          console.error("Error fetching topics from porzuczek.pl:", error);
          this.translatedTopics = topics;
        }
      } else {
        const target = this.selectedLanguage;
        const texts = topics.flatMap((topic) => [
          topic.title,
          ...topic.child.map((child) => child.title),
        ]);

        const url = `https://translation.googleapis.com/language/translate/v2?key=${this.apiKey}`;
        const data = {
          q: texts,
          target: target,
        };

        try {
          const response = await axios.post(url, data);
          const translations = response.data.data.translations.map(
            (t) => t.translatedText
          );

          let index = 0;
          this.translatedTopics = topics.map((topic) => {
            const translatedTopic = { ...topic, title: translations[index++] };
            translatedTopic.child = translatedTopic.child.map((child) => {
              return { ...child, title: translations[index++] };
            });
            return translatedTopic;
          });
        } catch (error) {
          console.error("Translation error:", error);
          this.translatedTopics = topics;
        }
      }
    },
    onItemClick(item) {
      this.$emit("select-content", item);
    },
    toggleSubmenu(index) {
      if (this.isSubmenuOpen(index)) {
        this.closeSubmenu(index);
      } else {
        this.openSubmenu(index);
      }
    },
    isSubmenuOpen(index) {
      return this.openSubmenus.includes(index);
    },
    openSubmenu(index) {
      this.openSubmenus.push(index);
    },
    closeSubmenu(index) {
      this.openSubmenus = this.openSubmenus.filter((i) => i !== index);
    },
    isTopicActive(topic) {
      return (
        (this.$parent.currentCourseTopicId &&
          topic.id === this.$parent.currentCourseTopicId &&
          !this.$parent.currentCourseQuestId) ||
        (topic.child &&
          topic.child.some(
            (child) => child.id === this.$parent.currentCourseQuestId
          ))
      );
    },
    isChildActive(child) {
      return (
        this.$parent.currentCourseQuestId &&
        child.id === this.$parent.currentCourseQuestId
      );
    },
    showNav() {
      this.isNavVisible = true;
    },
    hideNav() {
      this.isNavVisible = false;
    },
  },
};
</script>

<style scoped>
.navigation-container {
  position: fixed;
  top: 0;
  left: 0;
  height: 100%;
  z-index: 1000;
}

.navigation {
  width: 250px;
  background-color: var(--nav-bg-color);
  color: var(--nav-text-color);
  border-right: 1px solid #dee2e6;
  padding: 10px;
  box-shadow: 2px 0 5px rgba(0, 0, 0, 0.1);
  font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
  border-radius: 10px 0 0 10px;
  transform: translateX(-100%);
  transition: transform 0.3s ease-in-out;
}

.navigation-visible {
  transform: translateX(0);
}

.navigation-scroll {
  overflow-y: auto;
  max-height: 100vh;
}

.menu-item {
  margin-bottom: 10px;
}

.menu-title {
  font-weight: bold;
  cursor: pointer;
  padding: 5px 10px;
  transition: background-color 0.3s, color 0.3s;
  border-radius: 5px;
}

.menu-title:hover {
  background-color: var(--nav-hover-bg-color);
}

.submenu-item {
  margin-left: 20px;
}

.submenu-title {
  cursor: pointer;
  padding: 5px 10px;
  transition: background-color 0.3s, color 0.3s;
  border-radius: 5px;
}

.submenu-title:hover {
  background-color: var(--nav-hover-bg-color);
}
</style>
