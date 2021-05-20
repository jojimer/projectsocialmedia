<template>
  <q-page>
    <div class="u-post-box row">
      <div class="col-12">
          <div class="u-msgbox-comp">             
              <q-avatar class="q-ml-md">
                <img :src="currentUser.avatar">
              </q-avatar>
              <AudioPlayer
                v-if="audioPlayer"
                :uniqueID="'audioPostPlayback'"
                :AudioSRC="AudioSRC"
                :maxLength="maxLength"
                :closeBTN="true"
                :audioDuration="(duration) => audioDuration = duration"
                @close-player="resetRecording"
              />
              <div v-else class="u-message-box q-px-md">                
                  <InActiveBox
                    v-if="!recordingState"
                    :microphone="hasMicrophoneSupport"
                    :sound="soundMessage"
                    @chooseFiles="chooseFiles"
                    @captureAudio="captureAudio"
                  />
                  <ActiveBox
                    v-else
                    :microphone="hasMicrophoneSupport"
                    :sound="soundMessage"
                    :currentTime="currentTime"
                    :limit="limit"
                    @chooseFiles="chooseFiles"
                    @captureAudio="captureAudio"
                    @totalTime="getTotalTime"
                  />
            </div>
        </div>      
      </div>
      <div v-show="mentionBox" class="col-12">
        <div class="u-mention-bar-wrap">
          <div class="q-gutter-sm">
            <q-chip v-for="(user, index) in mentionedUser" :key="index">
              <q-avatar>
                <img :src="user.avatar">
              </q-avatar>
              {{user.handle}}
            </q-chip>
          </div>
        </div>
      </div>
      <div class="col-12">        
        <div class="u-btn-post-wrap">          
            <div class="u-post-mention">
              <q-btn
                class="q-mr-sm"
                color="primary"
                icon="fas fa-user-tag"
                label="Mention"
                flat
                no-caps
                rounded
                @click="postViewerWrap = false, postMentionWrap = !postMentionWrap"
              />
              <q-card  v-show="postMentionWrap" flat bordered class="post-mention-card absolute q-my-md q-mr-lg">
                <q-card-section class="q-pt-none">
                  <q-input v-model="searchMentionUser" @input="pushMentionOption" label="Search" />
                </q-card-section>
                <q-card-section v-show="searchMentionUser" class="q-pt-sm">
                  <div class="q-gutter-sm">
                    <q-chip v-for="(user, index) in mentionOption" :key="index">
                      <div style="cursor:pointer;" @click="pushMentionedUser(user)">
                        <q-avatar>
                          <img :src="user.avatar">
                        </q-avatar>
                        {{user.handle}}
                      </div>
                    </q-chip>
                  </div>
                </q-card-section>
              </q-card>
          </div>
          <div class="post-viewer-btn">
            <q-btn
              class="q-mr-md"
              color="primary"
              :icon="postViewer.icon"
              :label="postViewer.text+' can reply'"
              flat
              no-caps
              rounded
              @click="postViewerWrap = !postViewerWrap, postMentionWrap = false"
            />
            <q-card  v-show="postViewerWrap" flat bordered class="post-viewer-card absolute q-my-md q-mr-lg">
                <q-card-section>
                  <span class="text-subtitle2">Who can reply?</span>
                  <div class="text-caption text-grey-7">
                    Choose who can reply to this Post. Anyone mentioned can always reply.
                  </div>
                </q-card-section>
                <q-card-actions vertical class="q-px-none q-py-xs">
                  <q-btn @click="changePostViewer('Everyone')" color="primary" flat padding=".5rem 1rem" align="left" no-caps>
                    <span class="q-mr-auto"><q-icon class="q-mr-sm" :name="whoCanView.public.icon"/> <span class="text-grey-9">{{whoCanView.public.text}}</span></span>
                    <q-icon v-show="whoCanView.public.text == postViewer.text" name="check"/>
                  </q-btn>
                  <q-btn @click="changePostViewer('People you follow')" color="primary" flat padding=".5rem 1rem" align="left" no-caps>
                    <span class="q-mr-auto"><q-icon class="q-mr-sm" :name="whoCanView.following.icon"/> <span class="text-grey-9 q-mr-auto">{{whoCanView.following.text}}</span></span>
                    <q-icon v-show="whoCanView.following.text == postViewer.text" name="check"/>
                  </q-btn>
                  <q-btn @click="changePostViewer('Only people you mention')" color="primary" flat padding=".5rem 1rem" align="left" no-caps>
                    <span class="q-mr-auto"><q-icon class="q-mr-sm" :name="whoCanView.mention.icon"/> <span class="text-grey-9 q-mr-auto">{{whoCanView.mention.text}}</span></span>
                    <q-icon v-show="whoCanView.mention.text == postViewer.text" name="check"/>
                  </q-btn>
                </q-card-actions>
            </q-card>
          </div>          
          <q-btn :loading="posting" :disable="!audioPlayer && audioDuration === 0" @click="submitAudioPost" color="primary" dense rounded unelevated size="md" padding="xs lg" label="Post">
              <template v-slot:loading>
                <q-spinner-facebook
                  color="white"
                  size="1em"
                  :thickness="3"
                  class="on-center"
                />
              </template>
          </q-btn>
        </div>
      </div>
    </div>
    
    <AudioPost
      :posts="realPost"
      :users="dummyUser"      
    />

  </q-page>
</template>

<script>
import InActiveBox from '../components/RecorderBox/InActiveRecorder'
import ActiveBox from '../components/RecorderBox/ActiveRecorder'
import AudioPlayer from '../components/AudioPlayer'
import AudioPost from '../components/AudioPost'

export default {  
  name: 'PageIndex',
  data() {
    return {
      hasMicrophoneSupport: true,
      soundMessage: {
        message: '',
        file: ''
      },
      audioPlayer: false,
      AudioSRC: '',
      AudioBlob: '',
      AudioType: '',
      AUdioFrom: '',
      muted: false,
      media: null,
      recorder: '',
      recordingState: false,
      audioInputSrc: '',
      audioDuration: 0,
      currentTime: -1,
      limit: 60,     
      maxLength: '0:00',      
      postViewerWrap: false,
      postViewer: {
        icon: 'fas fa-globe-europe',
        text: 'Everyone'
      },
      whoCanView: {
        public: {
          icon: 'fas fa-globe-europe',
          text: 'Everyone'
        },
        following: {
          icon: 'fas fa-user-check',
          text: 'People you follow'
        },
        mention: {
          icon: 'alternate_email',
          text: 'Only people you mention'
        }
      },
      mentionBox: true,
      postMentionWrap: false,
      searchMentionUser: '',
      mentionedUser: [],
      mentionOption: [],
      audioPost: [],      
      realPost: [],
      posting: false,
      db: new this.$localbase('db')
    }    
  },
  props: {    
      currentUser: Object,
      dummyUser: Array,
  },
  components: {
    InActiveBox,
    ActiveBox,
    AudioPlayer,
    AudioPost
  },
  methods: {  
    async initMicrophone() {
      let items = []
      await navigator.mediaDevices.getUserMedia({
          audio: true,
          video: false
      }).then(stream => {        
          this.recorder = new MediaRecorder(stream)
          this.recorder.ondataavailable = e => {
            items.push(e.data)
            let blob = new Blob(items,{ 'type' : 'audio/ogg; codecs=opus' })
            this.AudioBlob = blob
            this.AudioSRC = URL.createObjectURL(blob)
            this.audioPlayer = true
            this.AUdioFrom = 'recording'
            if(this.recorder.state == 'inactive'){
              //alert('Done Recordeding')              
            }
          }
      }).catch(error => {
          this.hasMicrophoneSupport = false
          console.log(error)
      })
    },
    disableMicrophone() {
      if(this.recordingState){        
        this.recordingState = !this.recordingState
        this.recorder.stream.getAudioTracks().forEach(track => {
            track.stop()
        })
      }
    },
    async captureAudio() {
      if(!this.recordingState){
        await this.initMicrophone()
        await this.recorder.start()
        this.recordingState = !this.recordingState
        this.incrementTime()
      }else{
        this.recorder.stop()
        this.disableMicrophone()        
      }      
    },
    incrementTime() {
      let x = setInterval(() => {
          if(this.limit >= this.currentTime && this.recordingState){
            this.currentTime = this.currentTime+0.1
          }else if(this.recorder.state !== 'inactive' && this.recordingState){
            clearInterval(x)
            this.currentTime = 0
            this.recorder.stop()
            this.disableMicrophone()
            this.audioPlayer = true
          }else{
            clearInterval(x)
          }
        },100)

        //Use this if user change Tab
        // document.addEventListener("visibilitychange", event => {
        //   if (document.visibilityState == "visible") {
        //     console.log("tab is active")
        //   } else {
        //     console.log("tab is inactive")
        //   }
        // })
    },
    async resetRecording(val = true){      
      this.postMentionWrap = false
      this.postViewerWrap = false
      const reset = () => {
        this.audioPlayer = false
        this.currentTime = 0
        this.mentionedUser = []      
        this.posting = false
        this.postViewer = {
            icon: 'fas fa-globe-europe',
            text: 'Everyone'
        }    
      }
      if(val){
        setTimeout(() => {
          reset()    
          this.fetchPost()
        },3000)
      }else{
        reset()
      }      
    },
    getTotalTime(time){
      this.maxLength = time
    },
    changePostViewer(text){
      switch (text) {        
        case this.whoCanView.public.text: 
          this.postViewer = this.whoCanView.public  
        break
        case this.whoCanView.following.text: 
          this.postViewer = this.whoCanView.following
        break
        case this.whoCanView.mention.text: 
          this.postViewer = this.whoCanView.mention
        break
      }

      this.postViewerWrap = false
    },
    pushMentionOption() {
      let users = this.dummyUser
      let search = this.searchMentionUser
      let mentioned = this.mentionedUser
      let option = []
      if(mentioned.length > 0){
        mentioned.map(userM => {
          users.filter((user, index) => { if(user.handle === userM.handle) users.splice(index,1)})
        })        
      }
      users.map((user, index) => { 
        if(user.handle !== this.currentUser.handle){
          if(user.handle.toLowerCase().indexOf(search.toLowerCase()) !== -1) option.push(user)
        }else{
          users.splice(index,1)
        }        
      })
      this.mentionOption = option
    },
    pushMentionedUser(user) {
      this.mentionedUser.push(user)
      this.searchMentionUser = ''
    },
    chooseFiles() {
      let audioFile = document.querySelector("input#fileUpload")
      this.AUdioFrom = 'file'
      audioFile.click()
      //Add Event Listener to input audio file
      audioFile.addEventListener('change', (e) => {
        this.getAudioBlob(audioFile.files)
      },false)
      //console.log(audioFile)
    },
    convertDataURIToBinary(dataURI,BASE64_MARKER = ';base64,') {
      let base64Index = dataURI.indexOf(BASE64_MARKER) + BASE64_MARKER.length;
      let base64 = dataURI.substring(base64Index);
      let raw = window.atob(base64);
      let rawLength = raw.length;
      let array = new Uint8Array(new ArrayBuffer(rawLength));

      for(let i = 0; i < rawLength; i++) {
        array[i] = raw.charCodeAt(i);
      }
      return array;
    },
    getAudioBlob(files) {
      let reader = new FileReader()
      let type = files[0].type
      reader.readAsDataURL(files[0])
      reader.onload = (e) => {
        let binary = this.convertDataURIToBinary(reader.result)
        let blob = new Blob([binary],{ 'type' : type })
        this.AudioBlob = blob
        this.AudioSRC = URL.createObjectURL(blob)
        this.maxLength = 0
        this.audioPlayer = true
      }
    },
    async fetchPost() {
      let localDBpostlist = await this.db.collection('audiopost').orderBy('date', 'desc').get()
      this.realPost = await localDBpostlist
    },
    async submitAudioPost() {
      this.posting = true
      let length = (this.AUdioFrom === 'recording') ? this.maxLength : 0
      let newPostIDs = this.$props.currentUser.postIDs
      let userKey = 'user-key'+this.$props.currentUser.id
      let obj = {
        postID: Math.random(),
        user: this.currentUser,
        audioSRC: this.AudioSRC,
        audioBLOB: this.AudioBlob,
        audioLength: length,
        mentionedUsers: this.mentionedUser,
        viewer: this.postViewer,
        date: Date.now(),
      }      
      newPostIDs.push('post-key'+obj.postID)
      console.log(newPostIDs)
      await this.db.collection('audiopost').add(obj,'post-key'+obj.postID)
      await this.db.collection('users').doc(userKey).update({postIDs: newPostIDs})
      await this.resetRecording()
    },    
  },
  mounted() {      
    this.db.config.debug = false
    this.fetchPost()
  },
  created(){    
      
  },
  beforeDestroy() {
    if(this.hasMicrophoneSupport) {
      this.disableMicrophone()
    }
  },
}
</script>
<style scoped lang="sass">
  div.u-post-box
    border-bottom: .5px solid $grey-3
    padding-top: 51px
    padding-bottom: 24px


  div.u-msgbox-comp, div.u-message-box, div.u-message-control
    display: flex
    width: 100%
    justify-content: space-between

  div.u-media-player
    height: 60px !important
    width: 100%

  div.u-media-player div.q-media__controls
    padding: .5rem
    top: 10px

  div.u-media-player div.q-media__controls--standard div.q-media__controls--row
    margin-bottom: .5rem  

    
  div.u-btn-post-wrap, div.u-mention-bar-wrap
    display: flex
    justify-content: flex-end
    margin: 0.8em 1.5rem 0 auto
    width: 82%

  button.q-btn.disabled
    cursor: pointer!important

  div.post-mention-card, div.post-viewer-card
    width: 280px
    z-index: 1

</style>