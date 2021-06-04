<template>
    <div class="row q-pt-sm">
        <div class="col-7 q-px-md">
            <h6 class="q-ma-none q-pl-sm text-center text-weight-bold text-black" @click="friendsPagination">Friends</h6>
        </div>
        <div class="col-5 q-px-md">
            <q-input rounded outlined v-model="searchName" bg-color="grey-2" @input="filterFriends" dense label="Search" >
                <template v-slot:prepend>
                    <q-icon name="search" />
                </template>
            </q-input>
        </div>
        <div class="col-12 q-pt-md" style="border-bottom:1px solid #ccc"></div>        
        <div class="col-5 q-py-lg flex flex-center cursor-pointer q-mr-auto" @click="changeProfilePage('profile?u=',friend.handle.substring(1))" v-for="friend in friendsList" :key="friend.id">
                <q-avatar size="4rem">
                    <q-icon
                        v-if="friend.avatar === ''"
                        style="background-color:#f1f1f1;padding: 1.25rem;border-radius: 100%;"
                        name="far fa-user"
                        size="1.5rem"
                    />
                    <q-img
                        v-else
                        height="4rem"
                        :src="friend.avatar"
                        class="u-profile-picture"
                    />
                </q-avatar>
                <div class="inline q-ml-md">
                    <span class="text-subtitle1 text-primary block text-weight-bolder q-mt-md" style="line-height: 10px;">{{friend.name}}</span>
                    <span class="block text-caption text-grey-7 text-weight-regular">{{friend.handle}}</span>
                </div>
        </div>
        <div v-show="loading" class="col-12 text-center q-pa-xl">
            <q-spinner color="grey-3" size="2.5rem" :thickness="5" />
        </div>
    </div>
</template>

<script>
export default{
    data() {
        return {
            db: new this.$localbase('db'),
            dbName: this.$databaseName,
            friendsList: [],
            friendsInQueue: [],
            loadedFriends: [],
            allFriends: this.$props.friends.length,
            loading: false,
            searchName: '',
            pagination: 0,
            limit: 2,
            paginationStep: 2
        }
    },
    props: {
        friends: Array
    },
    methods: {
        getFriendsList(friends) {
            friends.map((friend, index) => {
                this.db.collection(this.dbName.users).doc((friend)).get().then(friend => {
                    this.friendsList.push(friend)
                    if(this.allFriends-1 === index){
                        this.loading = false
                    }
                })
            })            
        },
        friendsPagination() {
            this.loading = true
            let setFriends = []
            if(this.allFriends > this.pagination){
                for(this.pagination; this.pagination < this.limit; this.pagination++){
                    if(this.friendsInQueue[this.pagination]) setFriends.push(this.friendsInQueue[this.pagination])
                }
                this.limit = this.limit + this.paginationStep
                this.getFriendsList(setFriends) 
            }else{
                window.removeEventListener('scroll', this.friendsPagination)
                this.loading = false
            }
        },
        filterFriends() {
            let search = this.searchName
            this.loading = true 
            let filteredFriends
            this.pagination = 0
            this.limit = 2
            this.paginationStep = 2
            this.friendsList = []
            if(search){                
                filteredFriends = this.$props.friends.filter(friend => {
                    if(friend.name.toLowerCase().indexOf(search.toLowerCase()) !== -1){
                        return friend
                    }
                })
                
                this.allFriends = filteredFriends.length
                this.friendsInQueue = filteredFriends                
                this.getFriendsList(filteredFriends)
                console.log(search, this.friendsInQueue)
            }else{
                this.friendsInQueue = this.$props.friends
                this.allFriends = this.friendsInQueue.length                
                this.friendsPagination()
                window.addEventListener('scroll', this.friendsPagination)
            }
        },
        changeProfilePage(link,handle) {
            this.$router.push('profile?u='+handle)
            this.$emit('changeProfilePage',link+handle)
        }
    },
    created() {        
        this.friendsInQueue = this.$props.friends        
        window.addEventListener('scroll', this.friendsPagination)
        this.friendsPagination()
    },
    destroyed () {
        window.removeEventListener('scroll', this.friendsPagination)
    },
    mounted() {
        this.db.config.debug = false
    }
}
</script>