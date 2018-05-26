<template>
  <div id="app">
    <kanban-board :stages="stages" :blocks="blocks" @update-block="updateBlock">
      <div v-for="(block, index) in blocks" :slot="block.id">
    <div>
      <strong>id:</strong> {{ block.id }}
    </div>
    <div>
      {{ block.title }}
    </div>
  </div>
    </kanban-board>
  </div>
</template>

<script>
import Vue from "vue";
import vueKanban from "vue-kanban";
import Hamoni from "hamoni-sync";

Vue.use(vueKanban);

export default {
  name: "app",
  data() {
    return {
      stages: ["on-hold", "in-progress", "needs-review", "approved"],
      blocks: [],
      listPrimitive: null
    };
  },
  methods: {
    updateBlock(id, status) {
      let block = this.blocks[id];
      this.listPrimitive.update(id, { id, title: block.title, status });
    }
  },
  mounted: function() {
    let hamoni = new Hamoni("ACCOUNT_ID", "APP_ID");

    hamoni
      .connect()
      .then(() => {
        hamoni
          .get("board-12")
          .then(listPrimitive => {
            this.listPrimitive = listPrimitive;
            this.blocks = listPrimitive.getAll();
            listPrimitive.onItemUpdated(item => {
              this.blocks[item.index] = item.value;
            });
          })
          .catch(error => {
            if (error == "Error getting state from server") {
              hamoni
                .createList("board-12", blocks)
                .then(listPrimitive => {
                  this.listPrimitive = listPrimitive;
                  this.blocks = listPrimitive.getAll();
                  listPrimitive.onItemUpdated(item => {
                    this.blocks[item.index] = item.value;
                  });
                })
                .catch(console.log);
            }
          });
      })
      .catch(console.log);
  }
};

const blocks = [
  {
    id: 0,
    status: "approved",
    title: "Buy coffee machine"
  },
  {
    id: 1,
    status: "in-progress",
    title: "Find better AirBnB options"
  },
  {
    id: 2,
    status: "on-hold",
    title: "Approve Q3 budget"
  },
  {
    id: 3,
    status: "approved",
    title: "Travel to Guaraguara"
  },
  {
    id: 4,
    status: "needs-review",
    title: "Add Redux to the app"
  },
  {
    id: 5,
    status: "approved",
    title: "Well, Sleep all day 👩‍🎤"
  },
  {
    id: 6,
    status: "in-progress",
    title: "Find language exchange partner"
  }
];
</script>

<style lang="scss">
@import "./assets/kanban.scss";
</style>
