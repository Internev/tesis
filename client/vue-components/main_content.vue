<template>
  <div class="main-content">
    <navbar></navbar>

    <div>
    <!-- area to add live data as text is being added -->
     <div class="content-left">
     <div>
      <videocomponent id="video" :wsrtc="wsrtc" :uri="uri"></videocomponent>
     </div>

      <div class="doc-info">
        <div>{{ count }} words</div>
        <div>{{ time }} read</div>
      </div>
      <div class="audio">
        <audiocomponent id="audio" ></audiocomponent>
      </div>
      <div>
        <statcomponent :quill="quill"></statcomponent>
      </div>
    </div>
    <div class="content-right">
      <div id="editor"></div>
    </div>
  </div>
</template>

<script>
  import Navbar from './navbar.vue'
  import Methods from '../js/main_content.js'
  import Videocomponent from './video_component.vue'
  import Audiocomponent from './audio_component.vue'
  import {textStats, docSubscribe} from '../js/editor.js'
  import sharedb from 'sharedb/lib/client'
  import richText from 'rich-text'
  // import Quill from 'quill'
  import Chance from 'chance'
  import auth from '../js/auth.js'
  import docsave from '../js/docsave.js'
  import editor from '../js/editor.js'
  import statcomponent from './stat_component.vue'

  export default {
    created() {
      let chance = new Chance()
      let c = this.$route.params.channel
      const token = auth.getToken();
      this.uri = c !== undefined && /^\w{5}$/.test(c) ? c : chance.word({length: 5})
      //create RTC websocket
      this.wsrtc = new WebSocket(`wss://${window.location.host}/ws/${this.uri}rtc`);
      // update URL display. I still think we can do this with router somehow :S
      window.history.pushState(window.location.origin, '/', this.uri);
      // If token exists
      if (token) {
        // Checks if token in computer is valid then gets user resource
        auth.getJwt(this, token);
      }
    },
    mounted() {
      sharedb.types.register(richText.type)
      let socket = new WebSocket(`ws://${window.location.hostname}:3000/${this.uri}`)
      const connection = new sharedb.Connection(socket)
      // console.log(socket, this.wsrtc)
      // console.log(socket, this.wsrtc)
      // For testing reconnection
      window.disconnect = function() {
        connection.close();
      }
      window.connect = function(uri) {
        let socket = new WebSocket(`ws://${window.location.host}`);
        connection.bindToSocket(socket);
      }
      // Storing doc inside editor for access in other components.
      editor.doc = connection.get('docs', this.uri);
      // New quill
      editor.makeQuill();

      editor.quillOn(this, editor.doc);
      editor.docSubscribe(editor.quill, editor.doc);
      this.quill = editor.quill
    },
    data() {
      return {
        ws: null,
        wsrtc: null,
        channel: '',
        count: 0,
        // User data stored in auth
        user: auth.user,
        // Doc data stored in docsave
        docData: docsave.docData,
        time: '',
        quill: '',
        uri: ''
      }
    },
    components: {
      Navbar,
      Videocomponent,
      Audiocomponent,
      statcomponent
    },
    // Methods are located in js directory
    methods: Methods
  }
</script>

<style scoped>
.main-content{
  width: 100vw;
}
.content{
  display: inline-flex;
  height: 87vh;
  width: 100vw;
}
.content-right{
  width: 80%;
  display: inline-block;
  float: right;
}
.content-left{
  width: 20%;
  display: inline-block;
  justify-content: center;
  align-items: flex-end;
}
.doc-info{
  font-size: 0.95em;
  font-weight: 600;
  margin: 0.75em;
  opacity: 0.35;
}
html, body{
  color: #333;
  font-family: 'Monaco', courier, monospace;
}
#editor {
  height: 100%;
}
code {
  color: #f66;
}
</style>
