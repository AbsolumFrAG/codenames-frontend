<template>
  <div class="flex sm:my-6 w-full min-max-tab h-full sm:h-auto">
    <div
      :class="[
        'bg-main-300 sm:rounded-l-lg font-header text-white text-2xl p-4 w-16 flex flex-col justify-between',
        clickable,
      ]"
      @click="toggleChatTab()"
    >
      <Panel-icon
        icon="fa-comment-alt"
        margin="ml-1 mt-1"
        :showBadge="showChatBadge"
        :isTabOpen="isChatOpen"
        @set-badge="setChatBadge"
      />
      <i class="fas fa-times w-8 sm:hidden text-center mb-1"></i>
    </div>
    <div class="bg-main-100 w-full sm:w-80 xl:w-full flex flex-col">
      <h2
        class="
          w-full
          font-header
          text-white text-2xl
          px-6
          pl-4
          py-4
          flex
          items-center
          justify-start
        "
      >
        Chat de la partie
      </h2>
      <div ref="chatLog" class="overflow-y-auto pt-1 pb-0 flex-grow">
        <div v-for="msg in chatMessages" :key="msg.time">
          <div v-if="msg.systemMessage" class="text-sm">
            <p class="p-3 py-2 text-gray-200 tracking-wider">
              <i :class="`fas fa-${msg.icon} ml-1.5 mr-3`"></i>
              <span v-html="msg.content"></span>
            </p>
          </div>
          <div v-else class="text-sm">
            <p class="p-3 py-2 text-gray-200 tracking-wider flex items-start">
              <strong
                :class="`text-white flex-shrink-0 px-1.5 py-0.5 rounded-md truncate ${bgColor(
                  msg.sender.team
                )}`"
                >{{ msg.sender.name }}</strong
              >
              <span class="ml-3 mt-0.5 text-white break-words">{{
                msg.content
              }}</span>
            </p>
          </div>
        </div>
      </div>
      <div class="flex flex-nowrap items-center rounded-lg bg-main-200 m-3 p-2">
        <input
          type="text"
          v-model="chatMessage"
          ref="chatMessageField"
          @keypress.enter="sendMessage()"
          class="
            border-0
            py-2
            px-3
            text-white
            w-full
            focus:outline-none
            bg-transparent
          "
          maxlength="200"
          minlength="1"
          placeholder="Message"
        />
        <button
          @click="sendMessage()"
          title="Envoyer le message"
          class="
            focus:outline-none
            rounded-lg
            bg-main-300
            hover:bg-main-100
            text-white text-xl
            w-12
            h-10
            flex
            items-center
            justify-center
            mr-0.5
          "
        >
          <i class="fas fa-paper-plane"></i>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import PanelIcon from "../layout/PanelIcon.vue";

export default {
  name: "Chat",
  components: {
    PanelIcon,
  },
  data() {
    return {
      chatMessage: "",
      chatMessages: [],
    };
  },
  sockets: {
    chatMessage: function (data) {
      this.chatMessages.push(data);
      this.scrollChat();
    },
  },
  methods: {
    scrollChat() {
      this.$nextTick(() =>
        this.$refs.chatLog.scrollTo({
          top: this.$refs.chatLog.scrollHeight,
          behavior: "smooth",
        })
      );
    },
    sendMessage() {
      if (this.chatMessage.length > 0 && this.chatMessage.length <= 200) {
        this.$refs.chatMessageField.placeholder = "Message";
        this.$socket.emit("chatMessage", {
          roomCode: this.room.code,
          sender: this.player,
          content: this.chatMessage,
        });
        this.chatMessage = "";
        this.$refs.chatMessageField.focus();
        this.scrollChat();
      } else {
        this.$refs.chatMessageField.placeholder =
          "Le message ne peut pas être vide !";
      }
    },
    toggleChatTab() {
      this.$emit("toggle-chat-tab");
    },
    setChatBadge(value) {
      this.$emit("set-chat-badge", value);
    },
    bgColor: function (team) {
      if (team == "red") {
        return "bg-team-red";
      }
      if (team == "blue") {
        return "bg-team-blue";
      }
      if (team == "spectator") {
        return "bg-team-spectator";
      }
    },
  },
  computed: {
    clickable: function () {
      return this.$vssWidth < 1280 ? "cursor-pointer" : "";
    },
  },
  props: {
    room: {
      type: Object,
      required: true,
    },
    player: {
      type: Object,
      required: true,
    },
    showChatBadge: {
      type: Boolean,
      required: true,
    },
    isChatOpen: {
      type: Boolean,
      required: true,
    },
  },
  emits: ["toggle-chat-tab", "set-chat-badge"],
};
</script>

<style>
.min-max-tab {
  min-height: 320px;
  max-height: calc(100vh - 3rem);
}
@media only screen and (max-width: 640px) {
  .min-max-tab {
    min-height: 200px;
    max-height: 100%;
  }
}
</style>