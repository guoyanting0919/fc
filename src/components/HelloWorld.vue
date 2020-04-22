<template>
  <div>
    <div class="authentification">
      <h2>VueJS + Google Calendar Example</h2>Authentification
      <button v-if="!authorized" @click="handleAuthClick">Sign In</button>
      <button v-if="authorized" @click="handleSignoutClick">Sign Out</button>
    </div>
    <hr />
    <button v-if="authorized" @click="getData">Get Data</button>
    <div class="item-container" v-if="authorized && items">
      <pre v-html="items"></pre>
    </div>
  </div>
</template>

<script src="https://apis.google.com/js/api.js"></script>
<script>
export default {
  data() {
    return {
      items: undefined,
      api: undefined,
      authorized: false
    };
  },

  created() {},
  async mounted() {
    await console.log(gapi);
    this.api = gapi;
    this.handleClientLoad();
  },
  methods: {
    handleClientLoad() {
      this.api.load("client:auth2", this.initClient);
    },
    initClient() {
      let vm = this;
      vm.api.client
        .init({
          apiKey: "AIzaSyByY5wMUZ35A9lpgZqlLVJNSC3OFauCOEU",
          clientId:
            "1029849860188-2p6ecjh0egiiukqcn6cvesanckp4iqg0.apps.googleusercontent.com",
          discoveryDocs:
            "https://www.googleapis.com/discovery/v1/apis/calendar/v3/rest",
          scope: "https://www.googleapis.com/auth/calendar.readonly"
        })
        .then(_ => {
          // Listen for sign-in state changes.
          vm.api.auth2.getAuthInstance().isSignedIn.listen(vm.authorized);
        });
    },
    handleAuthClick(event) {
      Promise.resolve(this.api.auth2.getAuthInstance().signIn()).then(_ => {
        this.authorized = true;
        console.log(gapi);
      });
    },
    handleSignoutClick(event) {
      Promise.resolve(this.api.auth2.getAuthInstance().signOut()).then(_ => {
        this.authorized = false;
      });
    },
    getData() {
      let vm = this;

      vm.api.client.calendar.events
        .list({
          calendarId: "primary",
          timeMin: new Date().toISOString(),
          showDeleted: false,
          singleEvents: true,
          maxResults: 10,
          orderBy: "startTime"
        })
        .then(response => {
          vm.items = this.syntaxHighlight(response.result.items);
          console.log(vm.items);
        });
    },
    syntaxHighlight(json) {
      if (typeof json != "string") {
        json = JSON.stringify(json, undefined, 2);
      }
      json = json
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;");
      return json.replace(
        /("(\\u[a-zA-Z0-9]{4}|\\[^u]|[^\\"])*"(\s*:)?|\b(true|false|null)\b|-?\d+(?:\.\d*)?(?:[eE][+\-]?\d+)?)/g,
        match => {
          var cls = "number";
          if (/^"/.test(match)) {
            if (/:$/.test(match)) {
              cls = "key";
            } else {
              cls = "string";
            }
          } else if (/true|false/.test(match)) {
            cls = "boolean";
          } else if (/null/.test(match)) {
            cls = "null";
          }
          return '<span class="' + cls + '">' + match + "</span>";
        }
      );
    }
  }
};
</script>

<style>
body {
  font-family: "Open Sans", sans-serif;
  font-size: 14px;
  font-weight: 300;
  line-height: 1em;
}

.authentification {
  margin: 20px;
  text-align: center;
}

hr {
  border: 1px solid black;
  margin: 30px;
}

pre {
  outline: 1px solid #ccc;
  padding: 5px;
  margin: 5px;
  overflow-x: auto;
}

.string {
  color: green;
}

.number {
  color: purple;
}

.boolean {
  color: blue;
}

.null {
  color: magenta;
}

.key {
  color: black;
}
</style>