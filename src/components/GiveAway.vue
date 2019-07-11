<template>
  <div>
    <b-row>
      <b-col>
        <div class="custom-form">
          <b-form-input id="input1" v-model="keyword" placeholder="Keyword"></b-form-input>
        </div>
        <div class="custom-form">
          <b-form-select @change="setSubStatus()" v-model="substatus" :options="options"></b-form-select>
        </div>
        <b-row>
          <div class="custom-button">
            <b-button id="setKeyword" v-on:click="setKeyword()" variant="success">Start</b-button>
          </div>
          <div class="custom-button">
            <b-button id="ruffle" v-on:click="ruffle()" variant="danger">Ruffle</b-button>
          </div>
          <h4> Eligible : {{totalEligible}} / Total : {{totalUser}}</h4>
        </b-row>
        <nav>
          <b-list-group>
            <b-list-group-item v-for="(sub,user) in candidates" :key="user">
              <!-- <template v-if="sub[0] === 1">
                <label class="container">
                  {{user}}, (Subscribed {{sub[1]}} months)
                  <input
                    type="checkbox"
                    checked="checked"
                    disabled="disabled"
                  >
                  <span class="checkmark" v-on:click="toggle(user)"></span>
                </label>
              </template>
              <template v-else>
                <label class="container">
                  {{user}}, (Subscribed {{sub[1]}} months)
                  <input
                    type="checkbox"
                    disabled="disabled"
                  >
                  <span class="checkmark" v-on:click="toggle(user)"></span>
                </label>
              </template> -->
              <b-form-checkbox
                v-model="sub[0]"
                value=1
                unchecked-value=0
                @change="toggle(user)">
                 {{user}}, (Subscribed {{sub[1]}} months)
              </b-form-checkbox>
            </b-list-group-item>
          </b-list-group>
        </nav>
      </b-col>
      <b-col>
        <b-card header="Winners">
            <b-list-group>
                <b-list-group-item v-for="user in winners" :key="user[0]">
                    <h3>{{user[0]}}</h3>
                    Subscribed {{user[1]}} months
                </b-list-group-item>
            </b-list-group>
        </b-card>
      </b-col>
      <b-col>
        <iframe
          frameborder="0"
          scrolling="yes"
          src="https://www.twitch.tv/embed//chat"
          height="800"
          width="400"
        ></iframe>
      </b-col>
    </b-row>
  </div>
</template>

<script>
/* eslint-disable no-console */
/* eslint-disable no-undef */
/* eslint-disable no-unused-vars*/
import Vue from "vue";
import Vuex from "vuex";
Vue.use(Vuex);
const tmi = require("tmi.js");
const opts = {
  options: {
    clientId: ""
  },
  identity: {
    username: "",
    password: ""
  },
  channels: [""]
};
const client = new tmi.client(opts);
client.on("chat", onMessageHandler);
client.on("connected", onConnectedHandler);
var store = new Vuex.Store({
  state: {
    candidates: {
    },
    eligibles: new Set(),
    force: true
  }
});
var keyword = "";
var substatus = -1;
export default {
  name: "GiveAway",
  data: function() {
    return {
      keyword: "",
      substatus: -1,
      options: [
        { value: -1, text: "All users" },
        { value: 0, text: "All subscribers" },
        { value: 3, text: "Subscribers(3 or more)" },
        { value: 6, text: "Subscribers(6 or more)" },
        { value: 12, text: "Subscribers(12 or more)" }
      ],
      winners: [],
      totalEligible: 0
    }
  },
  computed: {
    candidates() {
      return store.state.candidates;
    },
    totalUser() {
      return Object.keys(this.candidates).length;
    },
    eligibles() {
      return store.state.eligibles;
    },
    force() {
      return store.state.force;
    }
  },
  watch: {
    force: function(newf,oldf) {
      this.totalEligible = this.eligibles.size;
    }
  },
  methods: {
    toggle: function(user) {
      if (this.candidates[user][0] === 1) this.eligibles.delete(user);
      else if (this.candidates[user][0] === 0) this.eligibles.add(user);
      Vue.set(this.candidates, user, [
        (this.candidates[user][0] + 1) % 2,
        this.candidates[user][1]
      ]);
      store.state.force = !store.state.force;
    },
    ruffle: function() {
      if (this.eligibles.size === 0) return;
      var rand = Math.floor(Math.random() * this.eligibles.size);
      var username = Array.from(this.eligibles)[rand];
      this.eligibles.delete(username);
      this.winners.push([username, store.state.candidates[username][1]]);
      Vue.set(this.candidates, username, [0, this.candidates[username][1]]);
      store.state.force = !store.state.force;
    },
    setKeyword: function() {
      keyword = this.keyword;
      store.state.candidates = {};
      store.state.eligibles = new Set();
      store.state.force = !store.state.force;
    },
    setSubStatus: function() {
      substatus = this.substatus;
      for (let user in this.candidates) {
        if (
          this.candidates[user][1] < substatus &&
          this.candidates[user][0] === 1
        ) {
          this.eligibles.delete(user);
          Vue.set(this.candidates, user, [0, this.candidates[user][1]]);
        }
        if (
          this.candidates[user][1] >= substatus &&
          this.candidates[user][0] === 0
        ) {
          this.eligibles.add(user);
          Vue.set(this.candidates, user, [1, this.candidates[user][1]]);
        }
      }
      store.state.force = !store.state.force;
    }
  }
};
function onMessageHandler(target, context, msg, self) {
  if (self || keyword === "") {
    return;
  }
  const commandName = msg.trim();
  if (commandName === keyword) {
    if (store.state.candidates[context["display-name"]]) return;
    if(!context["badges"]){
      if( substatus > -1){
        Vue.set(store.state.candidates, context["display-name"], [
        0,
        -1
      ]);
      }
      else{
        Vue.set(store.state.candidates, context["display-name"], [
        1,
        -1
      ]);
      store.state.eligibles.add(context["display-name"]);
      store.state.force = !store.state.force;
      }
    }
    else if(!context["badges"]["subscriber"]){
      if( substatus > -1){
        Vue.set(store.state.candidates, context["display-name"], [
        0,
        -1
      ]);
      }
      else{
        Vue.set(store.state.candidates, context["display-name"], [
        1,
        -1
      ]);
      store.state.eligibles.add(context["display-name"]);
      store.state.force = !store.state.force;
      }
    }
    else if (context["badges"]["subscriber"] >= substatus) {
      Vue.set(store.state.candidates, context["display-name"], [
        1,
        context["badges"]["subscriber"]
      ]);
      store.state.eligibles.add(context["display-name"]);
      store.state.force = !store.state.force;
    } else {
      Vue.set(store.state.candidates, context["display-name"], [
        0,
        context["badges"]["subscriber"]
      ]);
    }
  }
}
function onConnectedHandler(addr, port) {
  console.log(`* Connected to ${addr}:${port}`);
}
client.connect();
</script> 

<style scoped>
nav {
  width: 400px;
  height: 800px;
  overflow: hidden;
  overflow-y: scroll;
}

.custom-form {
  width: 300px;
  padding: 10px 5px 10px 0px;
}
.custom-button {
  padding: 10px 5px 10px 15px;
}

h4 {
  padding-top: 13px;
  padding-left: 25px;
}
</style>


