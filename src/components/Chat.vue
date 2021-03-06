<template>
  <div class="chat">
    <div class="chat-top">{{chatting.name}}</div>
    <div class="chat-main" ref="main">
      <div class="chat-main-item" v-for="(item, index) in chats" :key="index">
        <CompMessage :msg="item" />
        <span id="chat-main__tail" ref="tail"></span>
      </div>
    </div>
    <div class="chat-input">
      <el-input
        class="chat-input__area"
        v-model="msg"
        type="textarea"
        @keydown.enter.exact.native="handleEnter"
      ></el-input>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import Component from "vue-class-component";
import CompMessage from "@/components/Message.vue";
import { message } from "@/scripts/message";
import { contact } from "@/scripts/contact";
import { mapState } from "vuex";
import { user } from "@/scripts/user";
import { shh } from "../scripts/shh";
import { Watch } from "vue-property-decorator";

@Component({
  components: { CompMessage }
})
export default class Chat extends Vue {
  private msg: string = "";

  private get chatting(): contact.Group | contact.Private {
    return this.$store.state.chatting;
  }

  private get chats(): message.Message[] {
    return this.$store.state.chatLogs.get(this.chatting.id);
  }

  private get me(): user.User {
    return this.$store.state.user;
  }

  @Watch("chats") private handleChatsChange() {
    this.$nextTick(() => {
      this.scrollBottom();
    });
  }

  private async handleEnter() {
    if (!this.msg) {
      return;
    }
    const msgFix = this.msg.trim();
    if (!msgFix) {
      return;
    }

    if (this.chatting instanceof contact.Group) {
      const msg: message.Message = new message.Message(
        this.chatting.id,
        this.me.name,
        this.me.pubKey,
        msgFix,
        Date.now(),
        this.me.head
      );
      await this.$visitor.send(this.chatting.topic, msg);
    } else if (this.chatting instanceof contact.Private) {
      const msg: message.Message = new message.Message(
        "",
        this.me.name,
        this.me.pubKey,
        msgFix,
        Date.now(),
        this.me.head
      );
      await this.$visitor.sendPriv(this.chatting.pubKey, msg);
      msg.chatID = this.chatting.pubKey;
      this.$store.commit("pushMessage", msg);
    }

    setTimeout(() => {
      this.msg = "";
    }, 100);
  }

  private scrollBottom() {
    const comp: any = this.$refs.main;
    comp.scrollTop = comp.scrollHeight;
  }
}
</script>

<style lang="postcss" scoped>
.chat {
  height: 100%;
  display: flex;
  flex-direction: column;
  &-top {
    height: 40px;
    display: flex;
    font-weight: 600;
    align-items: center;
    justify-content: center;
    border-bottom: 1px solid var(--color-light-border);
  }
  &-main {
    flex: 1;
    overflow-y: auto;
  }
  &-input {
    height: 100px;
    border-top: 1px solid var(--color-light-border);
    &__area {
      height: 100%;
      & >>> .el-textarea__inner {
        border: none;
        height: 100%;
        resize: none;
      }
    }
  }
}
</style>