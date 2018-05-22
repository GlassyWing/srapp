<template>
  <v-layout row wrap>
    <v-flex xs12 sm6 class="px-2 py-2">
      <comp-recommend-panel ref="compRecommend"/>
    </v-flex>
    <v-flex xs12 sm4 class="px-2 py-2">
      <history-selector title="输入用户uuid和正在使用的构件名"
                        actionText="推荐"
                        @selected="refreshTables" ref="selector"/>
    </v-flex>
  </v-layout>
</template>

<script>
  import HistorySelector from "./historySelector";
  import CompRecommendPanel from "./compRecommendPanel";

  export default {
    name: "historyManage",
    components: {
      CompRecommendPanel,
      HistorySelector},
    data: () => ({
      search: false
    }),
    methods: {
      refreshTables(data) {
        this.$refs.compRecommend.getRecommendList(data.uuid, data.compName)
      }
    }
  }
</script>

<style scoped>

</style>
