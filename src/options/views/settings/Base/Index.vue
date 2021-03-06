<template>
  <div>
    <v-alert :value="true" type="info">{{ words.title }}</v-alert>
    <v-card color>
      <v-card-text>
        <v-form v-model="valid">
          <v-container fluid grid-list-xs>
            <v-layout row wrap>
              <v-flex xs12>
                <v-autocomplete
                  v-model="options.defaultClientId"
                  :items="this.$store.state.options.clients"
                  :label="words.defaultClient"
                  :menu-props="{maxHeight:'auto'}"
                  :hint="getClientAddress"
                  persistent-hint
                  item-text="name"
                  item-value="id"
                  required
                  :rules="rules.require"
                >
                  <template slot="selection" slot-scope="{ item }">
                    <span v-text="item.name"></span>
                  </template>
                  <template slot="item" slot-scope="data" style>
                    <v-list-tile-content>
                      <v-list-tile-title v-html="data.item.name"></v-list-tile-title>
                      <v-list-tile-sub-title v-html="data.item.address"></v-list-tile-sub-title>
                    </v-list-tile-content>
                    <v-list-tile-action>
                      <v-list-tile-action-text>{{ data.item.type }}</v-list-tile-action-text>
                    </v-list-tile-action>
                  </template>
                  <template slot="no-data">
                    <span class="ma-2">{{ words.noClient }}</span>
                  </template>
                </v-autocomplete>
              </v-flex>

              <v-flex xs5>
                <v-text-field
                  v-model="options.search.rows"
                  type="number"
                  :label="words.searchResultRows"
                  :placeholder="words.searchResultRows"
                ></v-text-field>
              </v-flex>
              <v-flex xs1>
                <v-slider v-model="options.search.rows" :max="200" :min="1"></v-slider>
              </v-flex>
              <v-flex xs6></v-flex>

              <v-flex xs5>
                <v-text-field
                  v-model="options.connectClientTimeout"
                  :label="words.connectClientTimeout"
                  :placeholder="words.connectClientTimeout"
                  type="number"
                ></v-text-field>
              </v-flex>
              <v-flex xs1>
                <v-slider
                  v-model="options.connectClientTimeout"
                  :max="60000"
                  :min="500"
                  :step="500"
                ></v-slider>
              </v-flex>

              <v-flex xs12>
                <!-- <v-switch
                  color="success"
                  v-model="options.autoUpdate"
                  :label="words.autoUpdate+lastUpdate"
                ></v-switch>-->
                <!-- 自动刷新用户数据 -->
                <v-switch
                  color="success"
                  v-model="options.autoRefreshUserData"
                  :label="words.autoRefreshUserData+autoRefreshUserDataLastUpdate"
                ></v-switch>

                <!-- 自动刷新用户数据时间 -->
                <v-flex xs12 v-if="options.autoRefreshUserData">
                  <div style="margin: -40px 0 10px 45px;">
                    <span>{{ words.autoRefreshUserDataTip1 }}</span>
                    <v-select
                      v-model="options.autoRefreshUserDataHours"
                      :items="hours"
                      class="mx-2 d-inline-flex"
                      style="max-width: 50px;max-height: 30px;"
                    ></v-select>
                    <span>:</span>
                    <v-select
                      v-model="options.autoRefreshUserDataMinutes"
                      :items="minutes"
                      class="mx-2 d-inline-flex"
                      style="max-width: 50px;max-height: 30px;"
                    ></v-select>
                    <span>{{ words.autoRefreshUserDataTip2 }}</span>
                  </div>
                </v-flex>

                <!-- 失败重试 -->
                <v-flex xs12 v-if="options.autoRefreshUserData">
                  <div style="margin: -20px 0 10px 45px;">
                    <span>{{ words.autoRefreshUserDataTip3 }}</span>
                    <v-select
                      v-model="options.autoRefreshUserDataFailedRetryCount"
                      :items="[1,2,3,4,5]"
                      class="mx-2 d-inline-flex"
                      style="max-width: 50px;max-height: 30px;"
                    ></v-select>
                    <span>{{ words.autoRefreshUserDataTip4 }}</span>
                    <v-select
                      v-model="options.autoRefreshUserDataFailedRetryInterval"
                      :items="[1,2,3,4,5]"
                      class="mx-2 d-inline-flex"
                      style="max-width: 50px;max-height: 30px;"
                    ></v-select>
                    <span>{{ words.autoRefreshUserDataTip5 }}</span>
                  </div>
                </v-flex>

                <!-- 启用内容选择搜索 -->
                <v-switch
                  color="success"
                  v-model="options.allowSelectionTextSearch"
                  :label="words.allowSelectionTextSearch"
                ></v-switch>

                <v-switch
                  color="success"
                  v-model="options.allowDropToSend"
                  :label="words.allowDropToSend"
                ></v-switch>

                <v-switch
                  color="success"
                  v-model="options.saveDownloadHistory"
                  :label="words.saveDownloadHistory"
                ></v-switch>

                <v-switch
                  color="success"
                  v-model="options.searchResultOrderBySitePriority"
                  :label="words.searchResultOrderBySitePriority"
                ></v-switch>

                <v-switch
                  color="success"
                  v-model="options.search.saveKey"
                  :label="words.saveSearchKey"
                ></v-switch>

                <v-switch
                  color="success"
                  v-model="options.showMoiveInfoCardOnSearch"
                  :label="words.showMoiveInfoCardOnSearch"
                ></v-switch>

                <!-- 在搜索之前一些选项配置 -->
                <v-switch
                  color="success"
                  v-model="options.beforeSearchingOptions.getMovieInformation"
                  :label="words.getMovieInformationBeforeSearching"
                ></v-switch>
                <v-flex xs12 v-if="options.beforeSearchingOptions.getMovieInformation">
                  <div style="margin: -40px 0 10px 45px;">
                    <span>{{ words.maxMovieInformationCount }}</span>
                    <v-text-field
                      v-model="options.beforeSearchingOptions.maxMovieInformationCount"
                      class="ml-2 d-inline-flex"
                      style="max-width: 100px;max-height: 30px;"
                      type="number"
                    ></v-text-field>
                    <v-slider
                      style="display:none;"
                      v-model="options.beforeSearchingOptions.maxMovieInformationCount"
                      :max="20"
                      :min="1"
                    ></v-slider>
                  </div>
                </v-flex>

                <!-- 当点击预选条目时，搜索模式 -->
                <v-flex xs12 v-if="options.beforeSearchingOptions.getMovieInformation">
                  <div style="margin: -20px 0 10px 45px;">
                    <span>{{ words.searchModeForItem }}</span>
                    <v-select
                      v-model="options.beforeSearchingOptions.searchModeForItem"
                      :items="searchModes"
                      class="mx-2 d-inline-flex"
                      style="max-height: 30px;"
                    ></v-select>
                  </div>
                </v-flex>

                <v-switch
                  color="success"
                  v-model="options.needConfirmWhenExceedSize"
                  :label="words.needConfirmWhenExceedSize"
                ></v-switch>

                <v-flex xs12 v-if="options.needConfirmWhenExceedSize">
                  <div style="margin: -40px 0 0 40px;">
                    <v-text-field
                      v-model="options.exceedSize"
                      :placeholder="words.exceedSize"
                      class="ml-2 d-inline-flex"
                      style="max-width: 100px;max-height: 30px;"
                    ></v-text-field>
                    <v-select
                      v-model="options.exceedSizeUnit"
                      :items="units"
                      class="mx-2 d-inline-flex"
                      style="max-width: 50px;max-height: 30px;"
                    ></v-select>
                  </div>
                </v-flex>
              </v-flex>
            </v-layout>
          </v-container>
        </v-form>
      </v-card-text>

      <v-divider></v-divider>

      <v-card-actions class="pa-3">
        <v-btn color="success" @click="save" :disabled="!valid">
          <v-icon>check_circle_outline</v-icon>
          <span class="ml-1">{{ words.save }}</span>
        </v-btn>
        <v-spacer></v-spacer>
        <!-- <v-btn color="warning" @click="clearCache">
          <v-icon>settings_backup_restore</v-icon>
          <span class="ml-1">{{ words.clearCache }}</span>
        </v-btn>-->
      </v-card-actions>
    </v-card>
    <v-snackbar v-model="haveError" absolute top :timeout="3000" color="error">{{ errorMsg }}</v-snackbar>
    <v-snackbar
      v-model="haveSuccess"
      absolute
      bottom
      :timeout="3000"
      color="success"
    >{{ successMsg }}</v-snackbar>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import { APP } from "@/service/api";
import {
  ESizeUnit,
  EAction,
  Options,
  EBeforeSearchingItemSearchMode
} from "@/interface/common";
import Extension from "@/service/extension";

const extension = new Extension();

export default Vue.extend({
  data() {
    return {
      words: {
        title: "常规设置",
        defaultClient: "默认下载服务器",
        autoUpdate: "自动更新官方数据",
        save: "保存",
        allowSelectionTextSearch: "启用页面内容选择搜索",
        allowDropToSend: "启用拖放链接到插件图标时，直接发送链接到下载服务器",
        clearCache: "清除缓存",
        clearCacheConfirm:
          "确认要清除缓存吗？清除完成后，下次将会从官网中重新下载系统配置信息。",
        needConfirmWhenExceedSize:
          "当批量下载的种子总体积超过以下大小时需要确认",
        exceedSize: "大小",
        searchResultRows: "搜索时每站点返回结果数量",
        saveDownloadHistory: "启用下载历史，以记录每次一键发送的种子信息",
        connectClientTimeout:
          "连接下载服务器超时时间（毫秒，1000毫秒=1秒），超出后将中断连接",
        noClient: "尚未配置下载服务器，请配置下载服务后再选择",
        cacheIsCleared: "缓存已清除，如需立即生效，请重新打开页面",
        saved: "参数已保存",
        autoRefreshUserData: "在浏览器打开的情况下自动刷新用户数据（Beta）",
        autoRefreshUserDataTip1: "每天于",
        autoRefreshUserDataTip2:
          "自动刷新（如果浏览器在这个时间之后打开，则在浏览器打开时自动刷新）",
        autoRefreshUserDataTip3: "失败后重试",
        autoRefreshUserDataTip4: "次，每次间隔",
        autoRefreshUserDataTip5: "分钟",
        searchResultOrderBySitePriority:
          "搜索结果点击站点表头时，按站点优先级别排序（保存后需刷新页面后生效）",
        saveSearchKey: "保存搜索关键字",
        showMoiveInfoCardOnSearch: "当以 IMDb 编号搜索时显示电影及评分信息",
        getMovieInformationBeforeSearching:
          "当输入搜索关键字时，从豆瓣加载相关信息以供预选",
        maxMovieInformationCount: "最多显示条目（1-20）：",
        searchModeForItem: "当点击预选条目时："
      },
      searchModes: [
        {
          value: EBeforeSearchingItemSearchMode.id,
          text: "以ID进行搜索，以获得较精确的内容，但转换ID时需要时间"
        },
        {
          value: EBeforeSearchingItemSearchMode.name,
          text: "以名称进行模糊搜索，以获得较多的内容"
        }
      ],
      valid: false,
      rules: {
        require: [(v: any) => !!v || "!"]
      },
      options: {
        defaultClientId: "",
        search: {
          saveKey: true
        },
        needConfirmWhenExceedSize: false,
        autoRefreshUserData: false,
        autoRefreshUserDataHours: "00",
        autoRefreshUserDataMinutes: "00",
        autoRefreshUserDataFailedRetryCount: 3,
        autoRefreshUserDataFailedRetryInterval: 5,
        connectClientTimeout: 10000,
        beforeSearchingOptions: {
          getMovieInformation: true,
          maxMovieInformationCount: 5,
          searchModeForItem: EBeforeSearchingItemSearchMode.id
        }
      } as Options,
      units: [] as any,
      hours: [] as any,
      minutes: [] as any,
      downloadHistory: [] as any,
      haveError: false,
      haveSuccess: false,
      successMsg: "",
      errorMsg: "",
      lastUpdate: "",
      autoRefreshUserDataLastUpdate: ""
    };
  },
  methods: {
    save() {
      console.log(this.options);
      this.$store.dispatch("saveConfig", this.options);
      this.successMsg = this.words.saved;
    },
    clearCache() {
      if (confirm(this.words.clearCacheConfirm)) {
        APP.cache.clear();

        setTimeout(() => {
          extension
            .sendRequest(EAction.reloadConfig)
            .then(() => {
              this.successMsg = this.words.cacheIsCleared;
            })
            .catch();
        }, 200);
      }
    }
  },
  created() {
    this.options = Object.assign(this.options, this.$store.state.options);
    this.units.push(ESizeUnit.MiB);
    this.units.push(ESizeUnit.GiB);
    this.units.push(ESizeUnit.TiB);
    this.units.push(ESizeUnit.PiB);

    for (let index = 0; index < 24; index++) {
      this.hours.push(`0${index}`.substr(-2));
    }

    for (let index = 0; index < 60; index += 5) {
      this.minutes.push(`0${index}`.substr(-2));
    }

    extension.sendRequest(EAction.getDownloadHistory).then((result: any) => {
      console.log("downloadHistory", result);
      this.downloadHistory = result;
    });
    APP.cache
      .getLastUpdateTime()
      .then((time: number) => {
        if (time > 0) {
          this.lastUpdate = `（最后更新于 ${new Date(time).toLocaleString()}）`;
        } else {
          this.lastUpdate = " （更新时间未知）";
        }
      })
      .catch(() => {
        this.lastUpdate = " （更新时间获取失败）";
      });

    if (this.options.autoRefreshUserDataLastTime) {
      this.autoRefreshUserDataLastUpdate = `（最后更新于 ${new Date(
        this.options.autoRefreshUserDataLastTime
      ).toLocaleString()}）`;
    }
  },
  watch: {
    successMsg() {
      this.haveSuccess = this.successMsg != "";
    },
    errorMsg() {
      this.haveError = this.errorMsg != "";
    }
  },
  computed: {
    getClientAddress(): any {
      if (!this.options.defaultClientId) {
        return "";
      }
      let client = this.$store.state.options.clients.find((data: any) => {
        return this.options.defaultClientId === data.id;
      });

      if (client) {
        return client.address;
      }
      return "";
    }
  }
});
</script>
<style lang="scss" scoped>
.v-input--selection-controls {
  margin: 0;
  padding: 0;
}
</style>
