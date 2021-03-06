<script>
import MChatTabs from "./chatTabs";
import { playTipSound } from "../util/play";

export default {
  name: "mchat",
  components: {
    MChatTabs,
  },
  provide() {
    return {
      rootChat: this,
    };
  },
  props: {
    config: {
      type: Object,
      default: () => ({
        rightBox: false,
        brief: false,
        voice: false,
        notice: false,
      }),
    },
    mine: {
      type: Object,
      default: () => ({
        id: "10001",
        username: "jule-meteor",
        status: "online",
        sign: "与其感慨路难行,不如马上出发！",
        avatar: "/avatar/avatar_meteor.png",
      }),
    },
    chats: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      panes: [],
      selected: "0",
      // 主体是否隐藏
      display: true,
      //chats是否隐藏
      chatDisplay: true,
      width: "800",
    };
  },
  computed: {
    alone() {
      let flag = true;
      if (this.chats.length > 1) {
        flag = false;
      }
      return flag;
    },
  },
  watch: {
    alone(nv, ov) {
      if (nv) {
        this.chatDisplay = true;
      }
    },
    "config.brief"(nv, ov) {
      if (!nv) {
        this.chatDisplay = true;
      }
    },
  },
  methods: {
    loadHistory(callBack) {
      this.$emit("loadHistory", callBack);
    },
    // 收到消息
    getMessage(message) {
      let voice = this.config.voice;
      // 提示音
      if (voice) {
        playTipSound();
      }
      this.panes.forEach((item) => {
        let { chat } = item;
        if (chat.id != message.id || chat.type != message.type) return;
        item.getMessage(message);
      });
    },
    handleEvent(event, data) {
      switch (event) {
        case "minRight":
          this.chatDisplay = !this.chatDisplay;
          break;
        case "tabClick":
          this.handleTabClick(data);
          break;
        case "tabRemove":
          this.handleRemoveChat(data);
          break;
        default:
          break;
      }
    },
    // 会话内容的事件
    bindTalkEvent(event, data) {
      this.$emit("talkEvent", event, data);
    },
    // 会话框的事件
    bindChatEvent(event, data) {
      this.$emit("chatEvent", event, data);
    },
    handleTabClick({ pane }) {
      this.selected = pane.index;
    },
    handleRemoveChat({ pane }) {
      const { name, type, id } = pane.chat;
      this.bindChatEvent("removeChat", { id, name, type });
    },
    handleEnter(chat, content) {
      const mine = this.mine;
      let message = {
        //自己的信息
        mine: {
          id: mine.id,
          username: mine.username,
          avatar: mine.avatar,
          mine: true,
        },
        // 目标
        to: {
          id: chat.id,
          name: chat.name,
          type: chat.type,
          avatar: chat.avatar,
        },
        //内容
        content,
        //数据类型
        type: "text",
        //发起的时间戳
        timestamp: new Date(),
      };
      //是否写回去

      this.$emit("sendMessage", message);
    },
    handPanesDrag(e) {
      let el = this.$refs.chat;
      var oDiv = el;
      var disX = e.clientX - oDiv.offsetLeft;
      var disY = e.clientY - oDiv.offsetTop;
      document.onmousemove = function(e) {
        e.preventDefault();
        var l = e.clientX - disX;
        var t = e.clientY - disY;
        oDiv.style.left = l + "px";
        oDiv.style.top = t + "px";
      };
      document.onmouseup = function() {
        document.onmousemove = null;
        document.onmouseup = null;
      };
    },
    //生成panes的数据
    calcPaneInstances(isForceUpdate = false) {
      //绕了一圈最终决定由chat-box 来决定 chat-tabs的属性
      if (this.$children) {
        const childPanes = this.$children.filter(
          (item) =>
            item.$vnode.tag &&
            item.$vnode.componentOptions &&
            item.$vnode.componentOptions.Ctor.options.name === "mchat-index"
        );
        // update indeed
        const panes = childPanes.map((item) => item.$vnode.componentInstance);
        const panesChanged = !(
          panes.length === this.panes.length &&
          panes.every((pane, index) => pane === this.panes[index])
        );
        let size = this.panes.length;
        if (isForceUpdate || panesChanged) {
          this.selected = "0";
          this.panes = panes;
        }
      } else if (size !== 0) {
        this.panes = [];
      }
    },
  },
  render() {
    let {
      handPanesDrag,
      config,
      chats,
      panes,
      bindTalkEvent,
      handleEvent,
      handleEnter,
      loadHistory,
      chatDisplay,
      alone,
    } = this;
    if (chats.length < 1) return;
    // 窗口页面
    const el_chat_panes = this._l(chats, (chat) => {
      let data_chat = {
        props: {
          chat,
          config,
        },
        ref: "MChatIndex",
        on: {
          enter: function(content) {
            handleEnter(chat, content);
          },
          loadHistory: function(callBack) {
            loadHistory(callBack);
          },
          talkEvent: function(event, data) {
            bindTalkEvent(event, data);
          },
        },
      };

      return <mchat-index {...data_chat}></mchat-index>;
    });

    // 标签页面
    const el_chat_tabs = {
      props: {
        panes,
      },
      ref: "MChatTabs",
      on: {
        click: function(event, data) {
          handleEvent(event, data);
        },
      },
    };
    return (
      <div>
        <div
          class={{
            "im-layer  layer-anim im-box im-chat": true,
            "chat-show": chatDisplay,
            alone: alone,
          }}
          ref="chat"
          style={{
            "z-index": 1002,
            left: "296.5px",
            display: "inline",
          }}
        >
          <div
            class="im-layer-title"
            style="cursor: move;"
            on-mousedown={(ev) => {
              handPanesDrag(ev);
            }}
          ></div>
          <div class="im-layer-tabs im-layer-content">
            <m-chat-tabs {...el_chat_tabs}></m-chat-tabs>
            {el_chat_panes}
          </div>
          <span class="im-box-setwin">
            <a class="im-btn-min" href="javascript:;">
              <cite></cite>
            </a>
            <a class="im-ico im-icon-max" href="javascript:;"></a>
            <a class="im-ico im-icon-close" href="javascript:;"></a>
          </span>
          <span class="im-icon-resize"></span>
        </div>
      </div>
    );
  },
  created() {
    // 设定一些监听的事件
    this.$im.on("getMessage", (msg) => {
      this.getMessage(msg);
    });
  },
  mounted() {
    this.calcPaneInstances();
  },
  updated() {
    this.calcPaneInstances();
  },
};
</script>

<style scoped>
.chat-show {
  -webkit-box-shadow: 1px 1px 10px rgba(0, 0, 0, 0.3);
  box-shadow: 1px 1px 10px rgba(0, 0, 0, 0.3);
  min-width: 500px;
  width: 800px;
}

.chat-show.alone {
  width: 620px;
}
</style>
