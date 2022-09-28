<template>
  <div class="row">
    <div class="col-8">
      <div class="card card-default">
        <div class="card-header">Messages</div>
        <div class="card-body p-0">
          <ul
            class="list-unstyled"
            style="height: 300px; overflow-y: scroll"
            v-chat-scroll
          >
            <!-- <li class="p-2" v-for="(message, index) in messages" :key="index">
              <strong>{{ message.user.name }}</strong>
              {{ message.message }}
            </li> -->

            <div class="chatbox" v-for="(message, index) in messages" :key="index">
              <div class="chat">
                  <strong>{{ message.user.name }}</strong>
                <img
                  src="https://images.unsplash.com/photo-1657299143482-4f4ea1ebd71c?ixlib=rb-1.2.1&ixid=MnwxMjA3fDF8MHxlZGl0b3JpYWwtZmVlZHwxMXx8fGVufDB8fHx8&auto=format&fit=crop&w=500&q=60"
                  alt=""
                />
                <p class="msg text-white">
                  {{ message.message }}
                </p>
              </div>
              <!-- <div class="chat">
                  <strong>{{ message.user.name }}</strong>
                <img
                  src="https://images.unsplash.com/photo-1660481451479-7ad6d6ad0223?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxlZGl0b3JpYWwtZmVlZHw3fHx8ZW58MHx8fHw%3D&auto=format&fit=crop&w=500&q=60"
                  alt=""
                />
                <p class="msg text-white">
                  {{ message.message }}
                </p>
              </div> -->
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
  },
};
</script>
<style scoped lang="scss">
// @import url("https://fonts.googleapis.com/css2?family=Open+Sans:wght@300;400;500;600;700&display=swap");

// * {
//   padding: 0;
//   margin: 0;
//   box-sizing: border-box;
//   font-family: open sans;
//   transition: ease 0.5s;
//   text-decoration: none;
//   list-style-type: none;
//   font-size: 12px;
//   font-weight: 300;
// }

// body {
//   background: #2f2d3f;
//   color: white;
// }

.container {
  width: 100%;
  max-width: 450px;
  background: #282639;
  margin: 0 auto;
  margin-top: 50px;
  border-radius: 10px;
  position: relative;
  box-shadow: #292739 0 10px 20px;
}

.header {
  background: #ff6969;
  display: flex;
  padding: 15px;
  border-radius: 10px 10px 0 0;
}
.grp-img {
  position: relative;
  width: 125px;
  height: 45px;
}
.grp-img img {
  width: 45px;
  height: 45px;
  border-radius: 50%;
  object-fit: cover;
  border: solid 2px white;
  position: absolute;
  top: 0;
  left: 0;
}
.grp-img .i1 {
  z-index: 1;
}
.grp-img .i2 {
  left: 40px;
}
.grp-img .i3 {
  left: 80px;
}
.grp-info {
  padding: 0 15px;
}
.grp-name {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: 3px;
}
.grp-status {
  letter-spacing: 1px;
}

.chatbox {
  padding: 40px 20px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}
.chatbox .chat {
  display: flex;
  gap: 15px;
}
.chatbox img {
  width: 45px;
  height: 45px;
  border-radius: 50%;
  object-fit: cover;
  border: solid 2px white;
}
.chatbox .msg {
  background: #3a384c;
  max-width: 80%;
  padding: 20px;
  border-radius: 10px;
  position: relative;
  font-size: 13px;
}
.msg ion-icon {
  font-size: 20px;
  color: #3a384c;
  position: absolute;
  top: 10px;
}
.chatbox .chat:nth-child(even) {
  flex-direction: row-reverse;
}
.chatbox .chat:nth-child(odd) ion-icon {
  left: -14px;
}
.chatbox .chat:nth-child(even) ion-icon {
  right: -14px;
  transform: rotate(180deg);
}

.policy {
  background: #3a384c;
  padding: 25px;
  border-radius: 0 0 10px 10px;
}
.policy p {
  font-size: 13px;
}
.policy p a {
  font-size: 13px;
  color: #ff6969;
  text-decoration: underline;
}
.agree {
  display: inline-block;
  color: #ff6969;
  border: solid 2px #ff6969;
  border-radius: 50px;
  padding: 10px 30px;
  font-size: 16px;
  font-weight: 500;
  margin-top: 20px;
}
.agree:hover {
  color: white;
  background: #ff6969;
}

.close {
  display: inline-block;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: #ff6969;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  position: absolute;
  bottom: -80px;
  right: 0;
}
.close:hover {
  box-shadow: #292739 0 0 10px 10px;
}
.close:hover ion-icon {
  transform: rotate(360deg);
}
.close ion-icon {
  font-size: 40px;
}
</style>