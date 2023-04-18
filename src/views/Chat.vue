<template>
  <div class="page__chat">
    <h1>Чат</h1>
    <router-link to="/">Домой</router-link>

    <div class="chat__messages">
      <transition name="loader">
        <Loader class="loader__img" v-if="loading" />
      </transition>
      <ul class="chat__messages-list" ref="messageBlock">
        <li class="chat__messages-item" v-for="(message, index) in messages" :key="index">{{ message }}</li>
      </ul>
    </div>
    <div><input type="text" v-model="message" placeholder="Введите сообщение" @keyup.enter="sendMessage"><button
        @click="sendMessage">Отправить</button></div>
    <transition name="error">
      <div class="error__block" v-if="error">Произошла ошибка <br>{{ error }}</div>
    </transition>
  </div>
</template>

<script>
import axios from "axios";
import Loader from "@/components/Loader.vue"
export default {
  data() {
    return {
      messages: [],
      page: 0,
      message: "",
      error: null,
      next: true,
      loading: false,
    };
  },
  methods: {
    sendMessage() {
      if (!this.message) return
      this.messages.push(this.message)
      this.message = "";
      this.$nextTick(this.scrollBlock)
    },
    async getMessages() {
      this.error = null;
      this.loading = true;
      try {
        const { data } = await axios.get(`https://numia.ru/api/getMessages?offset=${this.page}`)
        if (!this.messages.length) {
          this.messages = data.result?.reverse();
          this.$nextTick(this.scrollBlock);
        } else {
          if (data.result) { // так сделано потому что ошибка летит под кодом 200
            if (data?.result?.length) {
              const block = this.$refs.messageBlock;
              const windowHeight = block.scrollHeight;
              data.result.forEach((element) => {
                this.messages.unshift(element);
              });
              this.$nextTick(() => this.saveBlockPosition(windowHeight))
            } else {
              this.next = null;
            }
          } else {
            this.error = data;
            setTimeout(() => {
              this.error = null;
            }, 3000);
          }
        }
        this.page++;
      }
      catch (e) {
        console.error(e)
        this.error = e;
        setTimeout(() => {
          this.error = null;
        }, 3000);
      } finally {
        this.loading = false;
      }
    },
    saveBlockPosition(height) {
      const block = this.$refs.messageBlock;
      block.scrollTop = block.scrollHeight - height;
    },
    scrollBlock() {
      const block = this.$refs.messageBlock;
      block.scrollTo({
        top: block.scrollHeight,
        behavior: "smooth"
      })
    },
    scrolling() {
      const block = this.$refs.messageBlock;
      if (block.scrollTop === 0 && this.next !== null) {
        this.getMessages();
      }
    },
  },
  created() {
    this.getMessages();
  },
  mounted() {
    this.$refs.messageBlock.addEventListener("scroll", this.scrolling);
  },
  beforeUnmount() {
    this.$refs.messageBlock.removeEventListener("scroll", this.scrolling);
  },
  components: {
    Loader,
  },
}
</script>

<style>
.page__chat {
  max-width: 1200px;
  margin: 0 auto;
  display: grid;
}

.chat__messages {
  margin-bottom: 20px;
  margin-top: 20px;
  overflow: hidden;
}

.chat__messages-list {
  min-height: 400px;
  max-height: 500px;
  overflow-y: auto;
  border: 1px solid orange;
  border-radius: 6px;
  transition: all 5s ease-in-out;
  margin: 0;
}

.chat__messages-item {
  max-width: 300px;
  overflow-wrap: anywhere;
}

.error__block {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translateY(-50%) translateX(-50%);
  border: 3px solid red;
  background-color: #fff;
  z-index: 10;
  padding: 30px;
}

.loader__img {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
}

.loader-enter-active,
.loader-leave-active {
  transition: all 1s ease;
}

.loader-enter-from,
.loader-leave-to {
  opacity: 0;
}

.error-enter-active,
.error-leave-active {
  transition: all 1s ease;
}

.error-enter-from,
.error-leave-to {
  opacity: 0;
}
</style>