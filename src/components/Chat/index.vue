<script>
/**
 *  聊天的主体采用  rander 方式渲染
 *  带有业务耦合问题
 *  该页面是chat主页面 需要自己维护聊天信息
 */
// 聊天内容框
import chatList from "./chatList";
import enterBox from "./enterBox";
import tools from "./tools";

// 空方法

export default {
  name: "mchat-index",
  componentName: "MChatIndex",
  components: {
    chatList,
    enterBox,
    tools,
  },
  inject: ["rootChat"],
  props: {
    config: {
      type: Object,
      default: () => {},
    },
    chat: {
      type: Object,
      default: () => {},
    },
    // 工具栏配置
    toolConfig: {
      type: Object,
      default: () => ({
        show: ["file", "video", "img"],
        callback: Function,
      }),
    },
    height: {
      default: "500px",
    },
    width: {
      default: "550px",
    },
    //  输入属性 预留给上层模块
    value: {
      default: "",
    },
  },
  data() {
    return {
      // pane index
      index: "0",
      // 聊天记录
      taleList: [],
      //已加载
      loaded: true,
      // 滚动按钮
      scrollToButton: false,
      // 未读
      unread: 0,
      content: "",
      // 有边框控制器
      rightActive: false,
    };
  },
  computed: {
    // 是否被激活
    active() {
      const flag = this.rootChat.selected === this.index;
      return flag;
    },
  },
  watch: {
    // 滚动
    scroll(newVal) {
      if (typeof newVal === "number") {
        this.setScroll(newVal);
      }
    },
    value: {
      handler() {
        this.content = this.value;
      },
      immediate: true,
    },
    content: {
      handler() {
        this.$emit("input", this.content);
      },
      immediate: true,
    },
    scrollToButton(newVal) {
      if (newVal) {
        setTimeout(() => {
          this.scrollToButton = false;
        }, 0);
      }
    },
  },

  methods: {
    handleRighActive() {
      this.rightActive = !this.rightActive;
    },
    // 拉取历史记录
    loadHistory() {
      let that = this;
      //
      this.$emit("loadHistory", function (list) {
        list.forEach((item) => {
          that.taleList.unshift(item);
        });
      });
    },
    bindEnter(message) {
      this.$emit("enter", message);
    },
    // 处理收到的消息
    getMessage(message) {
      this.taleList.push(message);
    },
    bindEmoji(emoji) {
      this.content += emoji;
    },
    bindTalkEvent(event, data) {
      this.$emit("talkEvent", event, data);
    },
    handleUnread(count) {
      if (this.active) {
        this.unread = 0;
      } else {
        this.unread = count;
      }
    },
  },
  render(h) {
    let {
      rootChat,
      chat,
      config,
      active,
      taleList,
      rightActive,
      bindEmoji,
      bindTalkEvent,
      bindEnter,
      handleUnread,
      loadHistory,
      handleRighActive,
    } = this;
    let { name, avatar } = chat;

    let el_chat, el_chat_titel, data_chat_list, el_chat_footer;
    // 标题栏
    el_chat_titel = (
      <div class=" im-chat-title">
        <div class="im-chat-info" title="编辑群组">
          <img src={avatar} on-click={() => {}} />
          <span class="im-chat-username">{name}</span>
          <p class="im-chat-group">
            <span>群组</span>
          </p>
        </div>
      </div>
    );

    // chat list  的数据
    data_chat_list = {
      props: {
        list: taleList,
        config,
      },
      ref: "chatList",
      on: {
        messageUnread: function (count) {
          handleUnread(count);
        },
        loadHistory: function () {
          loadHistory();
        },
        talkEvent: function (event, data) {
          bindTalkEvent(event, data);
        },
      },
    };
    var self = this;
    let data_tools_bar = {
      props: {
        config,
      },
      on: {
        emoji: function (emoji) {
          bindEmoji(emoji);
        },
      },
    };

    let data_enter_box = {
      props: {
        value: self.content,
      },
      on: {
        input: function (value) {
          self.content = value;
        },
        submit: function (data) {
          bindEnter(data);
        },
      },
    };

    el_chat_footer = (
      <div
        class={{
          "im-chat-footer": true,
          listActive: rightActive,
        }}
      >
        <tools {...data_tools_bar}></tools>
        <enter-box {...data_enter_box}></enter-box>
      </div>
    );

    const el_right_box_slot = h("div", rootChat.$slots.default);

    // 挂载到该目标上
    const el_chat_right_box = (
      <div
        class={{
          "im-chat-right-box": true,
          display: rightActive,
        }}
      >
        {el_right_box_slot}
      </div>
    );
    let el_right_active = "";
    // 判断用户是否启用右边框
    if (rootChat.config.rightBox) {
      el_right_active = (
        <span
          class={{
            "im-chat-btn-expand": true,
            close: rightActive,
          }}
          on-click={() => handleRighActive()}
        >
          <i
            class={{
              "im-icon": true,
              "el-icon-arrow-left": !rightActive,
              "el-icon-arrow-right": rightActive,
            }}
          ></i>
        </span>
      );
    }

    el_chat = (
      <div
        class={{
          "im-chat-main": true,
          page: !rootChat.alone,
        }}
        id={`chat-${name}`}
        key={`chat-${name}`}
      >
        <div
          class={{
            "im-pane-item": true,
            display: active && rootChat.chatDisplay,
          }}
        >
          {el_chat_right_box}
          {el_chat_titel}
          <chat-list {...data_chat_list}> </chat-list>
          {el_chat_footer}
          {el_right_active}
        </div>
      </div>
    );
    return el_chat;
  },
};
</script>
