<template>
  <q-layout view="lHr lpR fFf">
    <q-header bordered class="bg-white text-black relative-p" style="z-index:2;width:65%;">
      <q-toolbar>
        <q-icon dense flat round name="menu" />

        <q-toolbar-title class="text-weight-bold">
          ASM
        </q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-drawer class="left-drawer" show-if-above v-model="left" side="left" bordered :width="283">
      <!-- <q-icon name="fas fa-circle" size="lg" style="height: 60px; max-width: 150px" color="negative" class="q-ma-md"/> -->
      <img src="logo/favicon-96x96.png" style="height: 60px; max-width: 60px" class="q-ma-md" />
      <q-list class="u-list-nav">
        <q-item class="u-link-rounded" clickable v-ripple to="/" @click="link = '/'" :active="link === '/'" :style="link !== '/' ? 'color:black;' : ''">
          <q-item-section avatar>
            <q-icon name="home" size="md" />
          </q-item-section>

          <q-item-section class="text-h6 text-weight-bold">Home</q-item-section>
        </q-item>

        <q-item class="u-link-rounded" clickable v-ripple :to="'/profile?u='+handle" @click="link = 'profile'" :active="link === 'profile'" :style="link !== 'profile' ? 'color:black;' : ''">
          <q-item-section avatar>
            <q-icon name="far fa-user" size="md" />
          </q-item-section>

          <q-item-section class="text-h6 text-weight-bold">Profile</q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <q-drawer
      class="right-drawer"
      v-model="right"
      side="right"
      bordered
      show-if-above
    >
      <q-input
        label="Search"
        class="q-ma-md fixed"
        outlined
        rounded
        dense
      >
        <template v-slot:prepend>
          <q-icon name="search" />
        </template>
      </q-input>
    <q-scroll-area class="fit">      

      <q-list padding separator style="margin-top:50px;">
        <q-item class="q-pa-md">
          <q-item-section>
            <q-item-label overline class="text-grey">OVERLINE</q-item-label>
            <q-item-label class="text-weight-bold">Single line item</q-item-label>
            <q-item-label caption>Secondary line text. Lorem ipsum dolor sit amet, consectetur adipiscit elit.</q-item-label>
          </q-item-section>

          <q-item-section side top>
            <q-item-label caption>5 min ago</q-item-label>
          </q-item-section>
        </q-item>

        <q-item class="q-pa-md">
          <q-item-section>
            <q-item-label overline class="text-grey">OVERLINE</q-item-label>
            <q-item-label class="text-weight-bold">Single line item</q-item-label>
            <q-item-label caption>Secondary line text. Lorem ipsum dolor sit amet, consectetur adipiscit elit.</q-item-label>
          </q-item-section>

          <q-item-section side top>
            <q-item-label caption>5 min ago</q-item-label>
          </q-item-section>
        </q-item>

        <q-item class="q-pa-md">
          <q-item-section>
            <q-item-label overline class="text-grey">OVERLINE</q-item-label>
            <q-item-label class="text-weight-bold">Single line item</q-item-label>
            <q-item-label caption>Secondary line text. Lorem ipsum dolor sit amet, consectetur adipiscit elit.</q-item-label>
          </q-item-section>

          <q-item-section side top>
            <q-item-label caption>5 min ago</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
    </q-scroll-area>
    </q-drawer>

    <q-page-container style="padding-top:0;">
      <router-view :currentUser="currentUser" :dummyUser="currentDummyUser" :link="link" v-on:changeLink="getActiveLink" />
    </q-page-container>
  </q-layout>
</template>

<script>
export default {
  data() {
    return {
      left: false,
      right: false,
      link: '',
      db: new this.$localbase('db'),
      handle: '',
      currentUser: {},
      currentDummyUser: [],
      dummyUser:[
        {
          id: '001',
          name: 'Quintino Araújo',
          handle: '@Quintino_Araújo',
          email: 'quintino.araujo@example.com',
          avatar: 'https://randomuser.me/api/portraits/men/53.jpg',
          postIDs: []
        },
        {
          id: '002',
          name: 'Nicole Jordan',
          handle: '@Nicole_Jordan',
          email: 'nicole.jordan@example.com',
          avatar: 'https://randomuser.me/api/portraits/women/64.jpg',
          postIDs: []
        },
        {
          id: '003',
          name: 'Manuela Gil',
          handle: '@Manuela_Gil',
          email: 'manuela.gil@example.com',
          avatar: 'https://randomuser.me/api/portraits/women/78.jpg',
          postIDs: []
        },
        {
          id: '004',
          name: 'Roy Brüning',
          handle: '@Roy_Brüning',
          email: 'roy.bruning@example.com',
          avatar: 'https://randomuser.me/api/portraits/men/73.jpg',
          postIDs: []
        },
        {
          id: '005',
          name: 'Christina Rogers',
          handle: '@Christina_Rogers',
          email: 'christina.rogers@example.com',
          avatar: 'https://randomuser.me/api/portraits/women/5.jpg',
          postIDs: []
        },
        {
          id: '006',
          name: 'Ronnie Welch',
          handle: '@Ronnie_Welch',
          email: 'ronnie.welch@example.com',
          avatar: 'https://randomuser.me/api/portraits/men/35.jpg',
          postIDs: []
        },
      ],
    };
  },
  props: {    
      //currentUser: Object
  },
  methods: {
    pickRandomUser() {
      //Select Random User
      let n = Math.floor((Math.random() * 5))
      this.currentUser = this.currentDummyUser[n]      
      this.handle = this.currentUser.handle.substring(1)
    },
    getActiveLink() {
      if(this.$route.path === 'profile' && this.$route.query.u === this.handle){
        this.link = 'profile'
      }else if(this.$route.path === '/'){
        this.link = '/'
      }else{
        this.link = ''
      }
    },
    goToProfile() {
      
    },
    async setUsers() { 
        this.dummyUser.map(user => {
          this.db.collection('users').add(user,'user-key'+user.id)
        })
        this.getUsers()
    },
    async getUsers() {
      try {
        let users = await this.db.collection('users').get()
        if(users.length === 0){
          this.setUsers()
        }else{
          this.currentDummyUser = users          
          this.pickRandomUser()    
          this.getActiveLink()
          //console.log(this.currentDummyUser)
        }
      }
      catch(error) {
        console.log('error: ', error)
      }
    }
  },
  mounted() {    
    this.db.config.debug = false    
    this.getUsers()
  }
};
</script>
