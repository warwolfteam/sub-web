<template>
  <div>
    <el-row style="margin-top: 10px">
      <el-col>
        <el-card>
          <div slot="header">
            订阅转换
            <!-- <div style="display: inline-block; position:absolute; right: 20px">{{ backendVersion }}</div> -->
          </div>
          <el-container>
            <el-form :model="form" label-width="120px" label-position="left" style="width: 100%">
              <el-form-item label="模式设置:">
                <el-radio v-model="advanced" label="1">基础模式</el-radio>
                <el-radio v-model="advanced" label="2">进阶模式</el-radio>
              </el-form-item>
              <el-form-item label="订阅链接:">
                <el-input
                  v-model="form.sourceSubUrl"
                  type="textarea"
                  rows="3"
                  placeholder="支持订阅或ss/ssr/vmess单链接。多个链接请每行一个或用 | 分隔"
                />
              </el-form-item>
              <el-form-item label="客户端:">
                <el-select v-model="form.clientType" style="width: 100%">
                  <el-option v-for="(v, k) in options.clientTypes" :key="k" :label="k" :value="v"></el-option>
                </el-select>
              </el-form-item>

              <el-form-item label="远程配置:">
                <el-select
                  v-model="form.remoteConfig"
                  allow-create
                  filterable
                  placeholder="请选择"
                  style="width: 100%"
                >
                  <el-option-group
                    v-for="group in options.remoteConfig"
                    :key="group.label"
                    :label="group.label"
                  >
                    <el-option
                      v-for="item in group.options"
                      :key="item.value"
                      :label="item.label"
                      :value="item.value"
                    ></el-option>
                  </el-option-group>
                </el-select>
              </el-form-item>

              <el-form-item label="后端地址:">
                <el-select
                  v-model="form.customBackend"
                  allow-create
                  filterable
                  placeholder="请选择"
                  style="width: 100%"
                >
                  <el-option v-for="(v, k) in options.customBackend" :key="k" :label="k" :value="v"></el-option>
                </el-select>
              </el-form-item>

              <div v-if="advanced === '2'">
                <el-form-item label="包含节点:">
                  <el-input v-model="form.includeRemarks" placeholder="节点名包含的关键字，支持正则" />
                </el-form-item>
                <el-form-item label="排除节点:">
                  <el-input v-model="form.excludeRemarks" placeholder="节点名不包含的关键字，支持正则" />
                </el-form-item>
                <el-form-item label="输出文件名:">
                  <el-input v-model="form.filename" placeholder="返回的订阅文件名" />
                </el-form-item>
                <el-form-item label-width="0px">
                  <el-row type="flex">
                    <el-col>
                      <el-checkbox v-model="form.emoji" label="Emoji" border></el-checkbox>
                      <el-checkbox v-model="form.nodeList" label="输出为 Node List" border></el-checkbox>
                    </el-col>
                    <el-popover placement="bottom" v-model="form.extraset">
                      <el-row>
                        <el-checkbox v-model="form.new_name" label="Clash New Field"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.udp" label="启用 UDP"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.appendType" label="节点类型"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.sort" label="排序节点"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.fdn" label="过滤非法节点"></el-checkbox>
                      </el-row>
                      <el-button slot="reference">更多选项</el-button>
                    </el-popover>
                    <el-popover placement="bottom" style="margin-left: 20px">
                      <el-row>
                        <el-checkbox v-model="form.tpl.surge.doh" label="Surge.DoH"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.tpl.clash.doh" label="Clash.DoH"></el-checkbox>
                      </el-row>
                      <el-row>
                        <el-checkbox v-model="form.insert" label="网易云"></el-checkbox>
                      </el-row>
                      <el-button slot="reference">定制功能</el-button>
                    </el-popover>
                  </el-row>
                </el-form-item>
              </div>

              <div style="margin-top: 50px"></div>

              <el-divider content-position="center">
                <i class="el-icon-magic-stick"></i>
              </el-divider>

              <el-form-item label="定制订阅:">
                <el-input class="copy-content" disabled v-model="customSubUrl">
                  <el-button
                    slot="append"
                    v-clipboard:copy="customSubUrl"
                    v-clipboard:success="onCopy"
                    ref="copy-btn"
                    icon="el-icon-document-copy"
                  >复制</el-button>
                </el-input>
              </el-form-item>

              <!-- <el-form-item label="订阅短链接:">
                <el-input class="copy-content" disabled v-model="curtomShortSubUrl">
                  <el-button
                    slot="append"
                    v-clipboard:copy="curtomShortSubUrl"
                    v-clipboard:success="onCopy"
                    ref="copy-btn"
                    icon="el-icon-document-copy"
                  >复制</el-button>
                </el-input>
              </el-form-item> -->

              <el-form-item label-width="0px" style="margin-top: 40px; text-align: center">
                <el-button
                  style="width: 120px"
                  type="danger"
                  @click="makeUrl"
                  :disabled="form.sourceSubUrl.length === 0"
                >生成订阅链接</el-button>
              </el-form-item>

              <el-form-item label-width="0px" style="text-align: center">
                <el-button
                  style="width: 120px"
                  type="primary"
                  @click="clashInstall"
                  icon="el-icon-connection"
                  :disabled="customSubUrl.length === 0"
                >一键导入Clash</el-button>
              </el-form-item>
            </el-form>
          </el-container>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
const defaultBackend = "https://doge.warwolfteam.cn/sub?";
// const defaultBackend = "https://api.wcc.best/sub?";
// const configUploadBackend = "https://api.wcc.best/config/upload";

export default {
  data() {
    var data = {
      // backendVersion: "",
      advanced: "1",

      // 是否为 PC 端
      isPC: true,

      options: {
        clientTypes: {
          Clash新参数: "clash&new_name=true",
          ClashR新参数: "clashr&new_name=true",
          Clash: "clash",
          ClashR: "clashr",
          Surge2: "surge&ver=2",
          Surge3: "surge&ver=3",
          Surge4: "surge&ver=4",
          Quantumult: "quan",
          QuantumultX: "quanx",
          Surfboard: "surfboard",
          Loon: "loon",
          ss: "ss",
          ssr: "ssr",
          ssd: "ssd",
          v2ray: "v2ray",
        },
        customBackend: {
          "localhost:25500 本地版": "http://localhost:25500/sub?",
          "本站": "https://doge.warwolfteam.cn/sub?",
        },
        backendOptions: [
          { value: "http://localhost:25500/sub?" },
          { value: "https://doge.warwolfteam.cn/sub?" },
        ],
        remoteConfig: [
          {
            label: "默认",
            options: [
              {
                label: "不选，由接口提供方提供",
                value: "",
              },
            ],
          },
          {
            label: "universal",
            options: [
              {
                label: "No-Urltest",
                value:
                  "https://subconverter.oss-ap-southeast-1.aliyuncs.com/Rules/RemoteConfig/universal/no-urltest.ini",
              },
              {
                label: "Urltest",
                value:
                  "https://subconverter.oss-ap-southeast-1.aliyuncs.com/Rules/RemoteConfig/universal/urltest.ini",
              },
            ],
          },
          {
            label: "customized",
            options: [
              {
                label: "Maying",
                value:
                  "https://subconverter.oss-ap-southeast-1.aliyuncs.com/Rules/RemoteConfig/customized/maying.ini",
              },
              {
                label: "rixCloud",
                value:
                  "https://subconverter.oss-ap-southeast-1.aliyuncs.com/Rules/RemoteConfig/customized/rixcloud.ini",
              },
              {
                label: "Nirvana",
                value:
                  "https://raw.githubusercontent.com/Mazetsz/ACL4SSR/master/Clash/config/V2rayPro.ini",
              },
              {
                label: "V2Pro",
                value:
                  "https://raw.githubusercontent.com/Mazeorz/airports/master/Clash/V2Pro.ini",
              },
              {
                label: "YoYu",
                value:
                  "https://subconverter.oss-ap-southeast-1.aliyuncs.com/Rules/RemoteConfig/customized/yoyu.ini",
              },
              {
                label: "Ytoo",
                value:
                  "https://subconverter.oss-ap-southeast-1.aliyuncs.com/Rules/RemoteConfig/customized/ytoo.ini",
              },
              {
                label: "NyanCAT",
                value:
                  "https://subconverter.oss-ap-southeast-1.aliyuncs.com/Rules/RemoteConfig/customized/nyancat.ini",
              },
              {
                label: "Nexitally",
                value:
                  "https://subconverter.oss-ap-southeast-1.aliyuncs.com/Rules/RemoteConfig/customized/nexitally.ini",
              },
              {
                label: "SoCloud",
                value:
                  "https://subconverter.oss-ap-southeast-1.aliyuncs.com/Rules/RemoteConfig/customized/socloud.ini",
              },
              {
                label: "ARK",
                value:
                  "https://subconverter.oss-ap-southeast-1.aliyuncs.com/Rules/RemoteConfig/customized/ark.ini",
              },
              {
                label: "ssrCloud",
                value:
                  "https://subconverter.oss-ap-southeast-1.aliyuncs.com/Rules/RemoteConfig/customized/ssrcloud.ini",
              },
              {
                label: "世葵Auto",
                value:
                  "https://raw.githubusercontent.com/Mazeorz/airports/master/Clash/SkslaPro-auto.ini",
              },
              {
                label: "世葵Balance",
                value:
                  "https://raw.githubusercontent.com/Mazeorz/airports/master/Clash/SkslaPro-Balance.ini",
              },
              {
                label: "贼船",
                value:
                  "https://subconverter.oss-ap-southeast-1.aliyuncs.com/Rules/RemoteConfig/customized/zeichuan.ini",
              },
              {
                label: "布丁",
                value:
                  "https://subconverter.oss-ap-southeast-1.aliyuncs.com/Rules/RemoteConfig/customized/pud.ini",
              },
            ],
          },
          {
            label: "Special",
            options: [
              {
                label: "NeteaseUnblock(仅规则，No-Urltest)",
                value:
                  "https://subconverter.oss-ap-southeast-1.aliyuncs.com/Rules/RemoteConfig/special/netease.ini",
              },
              {
                label: "Basic(仅GEOIP CN + Final)",
                value:
                  "https://subconverter.oss-ap-southeast-1.aliyuncs.com/Rules/RemoteConfig/special/basic.ini",
              },
            ],
          },
        ],
      },
      form: {
        sourceSubUrl: "",
        clientType: "",
        customBackend: "",
        remoteConfig: "",
        excludeRemarks: "",
        includeRemarks: "",
        filename: "",
        emoji: true,
        nodeList: false,
        extraset: false,
        sort: false,
        udp: false,
        tfo: false,
        scv: false,
        fdn: false,
        appendType: false,
        insert: false, // 是否插入默认订阅的节点，对应配置项 insert_url
        new_name: true, // 是否使用 Clash 新字段

        // tpl 定制功能
        tpl: {
          surge: {
            doh: false, // dns 查询是否使用 DoH
          },
          clash: {
            doh: false,
          },
        },
      },

      loading: false,
      customSubUrl: "",
      curtomShortSubUrl: "",

      dialogUploadConfigVisible: false,
      uploadConfig: "",
      uploadPassword: "",
      // myBot: tgBotLink,
      sampleConfig: "",
    };

    // window.console.log(data.options.remoteConfig);
    // window.console.log(data.options.customBackend);
    let phoneUserAgent = /Android|webOS|iPhone|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
      navigator.userAgent
    );

    if (phoneUserAgent) {
      let acl4ssrConfig = data.options.remoteConfig[1].options;
      for (let i = 0; i < acl4ssrConfig.length; i++) {
        if (acl4ssrConfig[i].label.length > 10) {
          acl4ssrConfig[i].label = acl4ssrConfig[i].label.replace(/\s.*/, "");
        }
      }
      var serverList = {};
      let serverKeys = Object.keys(data.options.customBackend);
      for (let i = 0; i < serverKeys.length; i++) {
        let key = serverKeys[i].replace(/\(.*/, "");
        serverList[key] = data.options.customBackend[serverKeys[i]];
      }
      data.options.customBackend = serverList;
    }
    return data;
  },
  created() {
    // document.title = "ACL4SSR Subscription Converter";
    document.title = "ACL4SSR 在线订阅转换";
    this.isPC = this.$getOS().isPc;
  },
  mounted() {
    this.form.clientType = "clash&new_name=true";
    this.form.customBackend = "https://doge.warwolfteam.cn/sub?";
    this.form.remoteConfig = "";
    this.notify();
    // this.getBackendVersion();
  },
  methods: {
    onCopy() {
      this.$message.success("Copied!");
    },
    clashInstall() {
      if (this.customSubUrl === "") {
        this.$message.error("请先填写必填项，生成订阅链接");
        return false;
      }

      const url = "clash://install-config?url=";
      window.open(
        url +
          encodeURIComponent(
            this.curtomShortSubUrl !== ""
              ? this.curtomShortSubUrl
              : this.customSubUrl
          )
      );
    },
    surgeInstall() {
      if (this.customSubUrl === "") {
        this.$message.error("请先填写必填项，生成订阅链接");
        return false;
      }

      const url = "surge://install-config?url=";
      window.open(url + this.customSubUrl);
    },
    makeUrl() {
      if (this.form.sourceSubUrl === "" || this.form.clientType === "") {
        this.$message.error("订阅链接与客户端为必填项");
        return false;
      }
      // 远程接口
      let backend =
        this.form.customBackend === ""
          ? defaultBackend
          : this.form.customBackend;

      // 远程配置
      let config = this.form.remoteConfig === "" ? "" : this.form.remoteConfig;

      let sourceSub = this.form.sourceSubUrl;
      sourceSub = sourceSub.replace(/(\n|\r|\n\r)/g, "|");

      // 薯条屏蔽
      if (
        sourceSub.indexOf("losadhwse") !== -1 &&
        (backend.indexOf("py6.pw") !== -1 ||
          backend.indexOf("subconverter-web.now.sh") !== -1 ||
          backend.indexOf("subconverter.herokuapp.com") !== -1 ||
          backend.indexOf("api.wcc.best") !== -1)
      ) {
        this.$alert(
          "此机场订阅已将该后端屏蔽，请自建后端转换。",
          "转换错误提示",
          {
            confirmButtonText: "确定",
            callback: (action) => {
              this.$message({
                type: "error",
                message: `action: ${action}`,
              });
            },
          }
        );
        return false;
      }

      this.customSubUrl =
        backend +
        "target=" +
        this.form.clientType +
        "&url=" +
        encodeURIComponent(sourceSub) +
        "&insert=" +
        this.form.insert;

      if (config !== "") {
        this.customSubUrl += "&config=" + encodeURIComponent(config);
      }

      if (this.advanced === "2") {
        if (this.form.excludeRemarks !== "") {
          this.customSubUrl +=
            "&exclude=" + encodeURIComponent(this.form.excludeRemarks);
        }
        if (this.form.includeRemarks !== "") {
          this.customSubUrl +=
            "&include=" + encodeURIComponent(this.form.includeRemarks);
        }
        if (this.form.filename !== "") {
          this.customSubUrl +=
            "&filename=" + encodeURIComponent(this.form.filename);
        }
        if (this.form.appendType) {
          this.customSubUrl +=
            "&append_type=" + this.form.appendType.toString();
        }

        this.customSubUrl +=
          "&emoji=" +
          this.form.emoji.toString() +
          "&list=" +
          this.form.nodeList.toString() +
          "&udp=" +
          this.form.udp.toString() +
          "&tfo=" +
          this.form.tfo.toString() +
          "&scv=" +
          this.form.scv.toString() +
          "&fdn=" +
          this.form.fdn.toString() +
          "&sort=" +
          this.form.sort.toString();

        if (this.form.tpl.surge.doh === true) {
          this.customSubUrl += "&surge.doh=true";
        }

        if (this.form.clientType === "clash") {
          this.customSubUrl += "&new_name=" + this.form.new_name.toString();
        }
      }

      this.$copyText(this.customSubUrl);
      this.$message.success("定制订阅已复制到剪贴板");
    },
    notify() {
      const h = this.$createElement;

      this.$notify({
        title: "隐私提示",
        type: "warning",
        message: h(
          "i",
          { style: "color: teal" },
          "各种订阅链接（短链接服务除外）生成纯前端实现，无隐私问题。默认提供后端转换服务，隐私担忧者请自行搭建后端服务。"
        ),
      });
    },
    backendSearch(queryString, cb) {
      let backends = this.options.backendOptions;

      let results = queryString
        ? backends.filter(this.createFilter(queryString))
        : backends;

      // 调用 callback 返回建议列表的数据
      cb(results);
    },
    createFilter(queryString) {
      return (candidate) => {
        return (
          candidate.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0
        );
      };
    },
    // getBackendVersion() {
    //   this.$axios
    //     .get(
    //       defaultBackend.substring(0, defaultBackend.length - 5) + "/version"
    //     )
    //     .then((res) => {
    //       this.backendVersion = res.data.replace(/backend\n$/gm, "");
    //       this.backendVersion = this.backendVersion.replace("subconverter", "");
    //     });
    // },
  },
};
</script>
