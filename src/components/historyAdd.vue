<template>
  <v-layout justify-center>
    <v-flex xs10>
      <v-card class="elevation-3">
        <v-card-title>
          <h3>添加一条历史纪录</h3>
        </v-card-title>
        <v-card-text>
          <v-form v-model="validate">
            <v-layout justify-start row wrap>
              <v-flex xs12>
                <v-text-field v-model="uuid" label="用户UUID" prepend-icon="perm_identity" :rules="uuidRules">
                </v-text-field>
              </v-flex>
              <v-flex xs12 class="py-3">
                <v-text-field v-model="compName" label="构件名" prepend-icon="extension" :rules="compNameRules">
                </v-text-field>
              </v-flex>
              <v-flex xs12 class="py-3">
                <v-text-field v-model="followCompName" label="下一个构件名" prepend-icon="extension" :rules="compNameRules">
                </v-text-field>
              </v-flex>
              <v-flex xs12 class="py-3">
                <v-btn color="primary" :disabled="!validate || !isCompNameExists || !isFollowCompNameExists"
                       @click="addHistory">添加历史纪录</v-btn>
              </v-flex>
            </v-layout>
          </v-form>
        </v-card-text>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
  import {COMP} from "../configs/srapp.api";
  import _ from 'lodash'

  export default {
    name: "history-add",
    data: () => ({
      validate: false,
      uuid: '',
      compName: '',
      isCompNameExists: false,
      followCompName: '',
      isFollowCompNameExists: false,
      isChecking: false,
      uuidRules: [
        v => !!v || 'uuid 不能为空',
        v => /^[0-9]+$/.test(v) || 'uuid 需满足数字格式'
      ],
      compNameRules: [
        v => !!v || '构件名不能为空',
        v => /^[A-Z]+\.[A-Za-z]+$/.test(v) || '构件名格式如：(ARRAYUTIL.join)'
      ]
    }),
    computed: {
      history() {
        return `${this.uuid},${this.compName},${this.followCompName}`
      }
    },
    methods: {
      checkIsCompExistsNow(compName, forCompName) {
        let query = compName.trim();
        if (!(!!query)) return;
        this.isChecking = true;
        this.$http.get(COMP + '/' + query)
          .then(response => {
            response.json().then(result => {
              let size = result['comps'].length;
              if (size !== 1) {
                this.$message.error("构件：" + compName + "不存在！");
                if (forCompName) {
                  this.isCompNameExists = false;
                } else {
                  this.isFollowCompNameExists = false;
                }
              } else {
                if (forCompName) {
                  this.isCompNameExists = true;
                } else {
                  this.isFollowCompNameExists = true;
                }
              }
              this.isChecking = false;
            }, error => {
              this.isChecking = false;
              this.$message.error("发生不可知的错误，请联系管理员!")
            })
          })
      },
      checkIsCompExistsWait: _.debounce(function () {
        this.checkIsCompExistsNow(this.compName, true)
      }, 500),
      checkIsFollowCompExistsWait: _.debounce(function () {
        this.checkIsCompExistsNow(this.followCompName, false)
      }, 500),
      addHistory() {
        let validateCount = 0;
        if (this.isChecking) {
          validateCount += 1;
          this.$message.info("正在检查中，请稍后！")
        }

        if (!this.isCompNameExists) {
          validateCount += 1;
          this.$message.warning("构件：" + this.compName + "不存在")
        }

        if (!this.isFollowCompNameExists) {
          validateCount += 1;
          this.$message.warning("下一个构件：" + this.followCompName + "不存在")
        }

        if (validateCount === 0) {
          console.log(this.history)
        }
      }
    },
    watch: {
      compName(newVal, oldVal) {
        this.checkIsCompExistsWait()
      },
      followCompName(newVal, oldVal) {
        this.checkIsFollowCompExistsWait()
      }
    }
  }
</script>

<style scoped>

</style>
