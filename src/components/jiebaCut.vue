<template>
  <v-card flat>
    <v-card-title>
      <h3>切分语句</h3>
    </v-card-title>
    <v-card-text>
      <v-text-field
        name="input-1"
        label="语句"
        id="statement"
        v-model="sentence"
      />


      <transition-group name="list" tag="p">
        <span v-for="word in words" :key="word" class="grey lighten-3 block">{{word}}</span>
      </transition-group>

    </v-card-text>
  </v-card>
</template>

<script>
  import {SEGMENT} from '../configs/srapp.api'

  export default {
    name: "jiebaCut",
    data: () => ({
      sentence: '',
      words: []
    }),
    watch: {
      sentence(newValue, oldValue) {
        if (!!newValue) {
          this.cut()
        } else {
          this.words = []
        }
      }
    },
    computed: {
      sentenceTrim() {
        return this.sentence.trim();
      }
    },
    methods: {
      cut: _.debounce(function () {
        if (!!this.sentenceTrim) {
          this.$http.post(SEGMENT + '?sentence=' + this.sentenceTrim + '&action=cut')
            .then(response => response.json().then(result => this.words = result['words'])
              , errors => this.$notify.error("发生未知错误，请联系管理员！"))
        }
      }, 500)
    }
  }
</script>

<style scoped>
  .block {
    display: inline-block;
    padding: 1px 4px;
    border-radius: 2px;
  }

  .list-item {
    display: inline-block;
    margin-right: 10px;
  }
  .list-enter-active, .list-leave-active {
    transition: all 0.5s;
  }
  .list-enter, .list-leave-to
    /* .list-leave-active for below version 2.1.8 */ {
    opacity: 0;
    transform: translateX(30px);
  }
</style>
