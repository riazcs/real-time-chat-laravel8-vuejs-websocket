<template>
  <div class="row">
    <div class="col-8">
      <div class="card card-default">
        <div class="card-header">Messages</div>
        <div class="card-body p-0">
          <ul
            class="list-unstyled"
            style="height: 400px; overflow-y: scroll"
            v-chat-scroll
          >
            <div v-for="(message, index) in messages" :key="'B' + index">
              <div :class="user.id === message.user.id ? 'container darker' : 'container'">
                <!-- <img
                src="https://www.w3schools.com/w3images/bandmember.jpg"
                alt="Avatar"
                style="width: 100%"
              /> -->
                <strong :class="user.id === message.user.id ? 'text-white time-right' : ''">{{ message.user.name }}</strong><span v-if="user.id === message.user.id"><br></span>
                <p :class="user.id === message.user.id ? 'text-white time-right' : ''">{{ message.message }}</p><span v-if="user.id === message.user.id"><br><br></span>
                 <span :class="user.id === message.user.id ? 'text-white time-left' : 'time-right'">{{
                  moment(message.created_at).format("LLLL")
                }}</span>
              </div>
            </div>
          </ul>
        </div>

        <input
          @keydown="sendTypingEvent"
          @keyup.enter="sendMessage"
          v-model="newMessage"
          type="text"
          name="message"
          placeholder="Enter your message..."
          class="form-control"
        />
      </div>
      <span class="text-muted" v-if="activeUser"
        >{{ activeUser.name }} is typing...</span
      >
    </div>

    <div class="col-4">
      <div class="card card-default">
        <div class="card-header">Active Users</div>
        <div class="card-body">
          <ul>
            <li class="py-2" v-for="(user, index) in users" :key="index">
              {{ user.name }}
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import moment from "moment";
export default {
  props: ["user"],

  data() {
    return {
      messages: [],
      newMessage: "",
      users: [],
      activeUser: false,
      typingTimer: false,
    };
  },

  created() {
    this.fetchMessages();

    Echo.join("chat")
      .here((user) => {
        this.users = user;
      })
      .joining((user) => {
        this.users.push(user);
      })
      .leaving((user) => {
        this.users = this.users.filter((u) => u.id != user.id);
      })
      .listen("MessageSent", (event) => {
        this.messages.push(event.message);
      })
      .listenForWhisper("typing", (user) => {
        this.activeUser = user;

        if (this.typingTimer) {
          clearTimeout(this.typingTimer);
        }

        this.typingTimer = setTimeout(() => {
          this.activeUser = false;
        }, 3000);
      });
  },

  methods: {
    fetchMessages() {
      axios.get("messages").then((response) => {
        this.messages = response.data;
      });
    },

    sendMessage() {
      this.messages.push({
        user: this.user,
        message: this.newMessage,
      });

      axios.post("messages", { message: this.newMessage });

      this.newMessage = "";
    },

    sendTypingEvent() {
      Echo.join("chat").whisper("typing", this.user);
    },
    moment() {
      return moment();
    },
  },
};
</script>
<style scoped lang="scss">
body {
  margin: 0 auto;
  max-width: 800px;
  padding: 0 20px;
}

.container {
  border: 2px solid #dedede;
  background-color: #f1f1f1;
  border-radius: 5px;
  padding: 10px;
  margin: 10px 0;
}

.darker {
  border-color: #ccc;
  background-color: rgb(84, 83, 83);
}

.container::after {
  content: "";
  clear: both;
  display: table;
}

.container img {
  float: left;
  max-width: 60px;
  width: 100%;
  margin-right: 20px;
  border-radius: 50%;
}

.container img.right {
  float: right;
  margin-left: 20px;
  margin-right: 0;
}

.time-right {
  float: right;
  color: #aaa;
}

.time-left {
  float: left;
  color: #999;
}
</style>