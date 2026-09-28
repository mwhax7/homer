<template>
  <Generic :item="item">
    <template #content>
      <p class="title is-4">{{ item.name }}</p>
      <p class="subtitle is-6">
        <template v-if="item.subtitle">
          {{ item.subtitle }}
        </template>
        <template v-else-if="status === 'running'">
          {{ software }} | v{{ version }} | {{ players.online }}/{{ players.max }} players
        </template>
      </p>
    </template>
    <template #indicator>
      <div v-if="status" class="status" :class="status">
        {{ status }}
      </div>
    </template>
  </Generic>
</template>

<script>
import service from "@/mixins/service.js";

export default {
  name: "Minecraft",
  mixins: [service],
  props: {
    item: Object,
  },
  data: () => ({
    status: "",
    software: "",
    version: "",
    players: {
      online: 0,
      max: 0,
    },
  }),
  created() {
    // Set up auto-update method for the scheduler
    this.autoUpdateMethod = this.fetchServerStatus;

    // Initial data fetch
    this.fetchServerStatus();
  },
  methods: {
    fetchServerStatus: async function () {
      if (!this.server) {
        console.error(
          `Minecraft: "${this.item.name}" is missing the url option`,
        );
        this.status = "error";
        return;
      }

      try {
        const data = await this.fetch(this.server);

        if (!data.online) {
          this.status = "stopped";
          return;
        }

        this.status = "running";
        // Both are optional and free form: plenty of servers report neither.
        this.software = data.software || "";
        this.version = data.version || "";
        this.players.online = data.players?.online || 0;
        this.players.max = data.players?.max || 0;
      } catch (e) {
        console.error(e);
        this.status = "error";
      }
    },
  },
};
</script>

<style scoped lang="scss">
.status {
  font-size: 0.8rem;
  color: var(--text-title);

  &.running:before {
    background-color: #94e185;
    border-color: #78d965;
    box-shadow: 0 0 5px 1px #94e185;
  }

  &.stopped:before,
  &.error:before {
    background-color: #c9404d;
    border-color: #c42c3b;
    box-shadow: 0 0 5px 1px #c9404d;
  }

  &:before {
    content: " ";
    display: inline-block;
    width: 7px;
    height: 7px;
    margin-right: 10px;
    border: 1px solid #000;
    border-radius: 7px;
  }
}
</style>
