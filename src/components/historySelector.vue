<template>
  <v-card flat>
    <v-card-title>
      <h3>{{title}}</h3>
    </v-card-title>
    <v-card-text>
      <v-form v-model="validate">
        <v-layout v-if="!horizon" justify-start row wrap>
          <v-flex xs12>
            <v-text-field v-model="uuid" label="用户UUID" prepend-icon="perm_identity" :rules="uuidRules">
            </v-text-field>
          </v-flex>
          <v-flex xs12 class="py-3">
            <v-text-field v-model="compName" label="构件名" prepend-icon="extension" :rules="compNameRules">
            </v-text-field>
          </v-flex>
          <v-flex xs12 class="py-3">
            <v-btn color="primary" @click="sendSignal" :disabled="!validate || !isCompNameExists">{{actionText}}</v-btn>
          </v-flex>
        </v-layout>

        <v-layout v-if="horizon" justify-center>
          <v-flex xs12 sm4>
            <v-text-field v-model="uuid" label="用户UUID" prepend-icon="perm_identity" :rules="uuidRules">
            </v-text-field>
          </v-flex>
          <v-flex xs12 sm4>
            <v-text-field v-model="compName" label="构件名" prepend-icon="extension" :rules="compNameRules">
            </v-text-field>
          </v-flex>
          <v-flex xs12 sm4>
            <v-btn color="primary" @click="sendSignal" :disabled="!validate || !isCompNameExists">{{actionText}}</v-btn>
          </v-flex>
        </v-layout>
      </v-form>
    </v-card-text>
  </v-card>

</template>

<script>
  import {COMP} from "../configs/srapp.api";
  import _ from 'lodash'

  export default {
    name: "historySelector",
    props: {
      title: '',
      actionText: '',
      horizon: false
    },
    data: () => ({
      validate: false,
      addable: false,
      uuid: '',
      compName: '',
      isCompNameExists: false,
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
    methods: {

      sendSignal() {
        let validateCount = 0;
        if (this.isChecking) {
          validateCount += 1;
          this.$message.info("正在检查中，请稍后！")
        }

        if (!this.isCompNameExists) {
          validateCount += 1;
          this.$message.warning("构件：" + this.compName + "不存在")
        }

        if (validateCount === 0) {
          this.$emit("selected", {uuid: this.uuid, compName: this.compName})
        }
      },
      checkIsCompExistsNow(compName) {
        let query = compName.trim();
        if (!(!!query)) return;
        this.isChecking = true;
        this.$http.get(COMP + '/' + query)
          .then(response => {
            response.json().then(result => {
              let size = result['comps'].length;
              if (size !== 1) {
                this.$message.error("构件：" + compName + "不存在！");
                this.isCompNameExists = false;
              } else {
                this.isCompNameExists = true;
              }
              this.isChecking = false;
            }, error => {
              this.isChecking = false;
              this.$message.error("发生不可知的错误，请联系管理员!")
            })
          })
      },
      checkIsCompExistsWait: _.debounce(function () {
        this.checkIsCompExistsNow(this.compName)
      }, 600)
    },

    watch: {
      compName(newVal, oldVal) {
        this.checkIsCompExistsWait()
      }
    }
  }
</script>

<style scoped>
  .radius {
    border-radius: 5px;
    border: 1px;
  }
</style>
