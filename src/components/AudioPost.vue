<template>
    <div v-if="this.$props.posts.length > 0" class="u-all-post-box q-py-sm q-mb-md">
        <div class="u-audio-post-box q-ma-none q-pt-md" v-for="(post, index) in this.$props.posts" :key="post.key">
            <div class="q-pa-sm q-mb-none row">
                <q-avatar class="q-ml-md q-mt-md">
                    <img :src="post.data.user.avatar">
                </q-avatar>
                <div class="u-user-name-handle col-7">
                    <router-link class="text-subtitle2 text-bold text-primary" :to="'/profile?u='+post.data.user.handle.substring(1)">{{post.data.user.name}}</router-link>
                    <div class="text-subtitle2 text-grey-8 text-weight-regular">{{post.data.user.handle}}</div>
                </div>
                <div class="text-subtitle2 text-grey col-3 u-post-date text-right">
                    {{ post.data.date | niceDate }}
                </div>
                <!-- <div class="col-2">
                    <q-icon :name="post.viewer.icon"></q-icon>
                    {{post.viewer.text+' can reply'}}
                </div> -->
            </div>
            <AudioPlayer
                :uniqueID="'userIndex'+index"
                :AudioSRC="getAudioSource(post.data.audioBLOB)"
                :maxLength="post.data.audioLength"
                :closeBTN="false"
            />
            <div class="u-post-mentioned">
                <router-link :to="'profile?u='+user.handle.substring(1)" v-for="(user, index) in post.data.mentionedUsers" :key="index">
                <q-chip class="q-mt-md q-mb-none" @click="this.$emit('changeProfilePage')">
                    <q-avatar>
                        <img :src="user.avatar">
                    </q-avatar>
                    {{user.handle}}
                </q-chip>
                </router-link>
            </div>
            <div class="u-post-action q-mt-lg">
                <q-btn-group flat spread>
                    <q-btn
                        @click="toggleLikes(post.key,post.data.likes)"
                        class="q-py-sm u-like-react"
                        :color="checkIfUSerLikeThePost(post.key) ? 'red-5' : 'grey-5'"
                        :icon="checkIfUSerLikeThePost(post.key) ? 'fas fa-heart' : 'far fa-heart'"
                        :label="post.data.likes.length > 0 ? post.data.likes.length : ''"
                        flat
                    />
                    <q-btn
                        class="q-py-sm u-comment"
                        color="grey-5"
                        icon="far fa-comment-alt"
                        label="Reply"
                        flat
                    />
                    <q-btn
                        class="q-py-sm u-repost"
                        color="grey-5"
                        icon="fas fa-retweet"
                        flat
                    />
                </q-btn-group>
            </div>
        </div>
    </div>
    <div v-else-if="loading" style="margin:7rem auto; width:20%; text-align:center;">
        <q-spinner
            color="primary"
            size="3rem"
            :thickness="3"
        />
    </div>
    <div v-else class="q-py-lg q-mb-md text-center text-grey-5">
        <h6 class="text-weight-regular">No post available!</h6>
    </div>
</template>

<script>
import AudioPlayer from './AudioPlayer'
import { date } from "quasar"

export default {
    data() {
        return {
            muted: true,
            likeColor: [],
            loading: true,
            db: new this.$localbase('db'),
            dbName: this.$databaseName
        }
    },
    props: {
        posts: Array,
        users: Array,
        userID: String,
        userLikes: Array
    },
    components: {
        AudioPlayer
    },
    methods: {
        getAudioSource(blob){
            return URL.createObjectURL(blob)
        },
        async toggleLikes(key,likers){
            let userID = this.$props.userID
            let userlikes = this.$props.userLikes            
            
            if(!userlikes.includes(key) && !likers.includes(userID)){
                likers.push(userID)
                userlikes.push(key)
                await this.db.collection(this.dbName.audiopost).doc(key).update({likes: likers})
                await this.db.collection(this.dbName.users).doc('user-key'+userID).update({likes: userlikes})
                //console.log('Add')
            }else{                
                const rlikers = likers.indexOf(userID)
                const rpostKey = userlikes.indexOf(key)
                if (rlikers > -1 && rpostKey > -1) {
                    //Remove likes from post
                    likers.splice(rlikers, 1)
                    await this.db.collection(this.dbName.audiopost).doc(key).update({likes: likers})
                    //Remove post like to user data
                    userlikes.splice(rpostKey, 1)
                    await this.db.collection(this.dbName.users).doc('user-key'+userID).update({likes: userlikes})
                }
                //console.log('Delete')
            }            
        },
        checkIfUSerLikeThePost(key){
            let userlikes = this.$props.userLikes
            const rpostKey = userlikes.indexOf(key)
            return (rpostKey > -1) ? true : false
        }
    },
    created() {
        this.db.config.debug = false
        setTimeout(() => {
            this.loading = false
            //console.log(this.$props.posts)
        },3000)
    },
    filters: {
        niceDate(time) {
            return date.formatDate(time, "MMMM D, h:mmA ");
        },
    },
    watch: {
        posts() {
            setTimeout(() => {
                this.loading = false
            },3000)
        }
    }
}
</script>
<style scoped lang="sass">
    div.u-all-post-box
        background-color: $grey-1

    div.u-audio-post-box
        height: 100%
        background-color: white
        border-top: .5px solid $grey-3

    div.u-audio-post-box div.u-audio-player
        width: 85%
        margin-left: auto

    div.u-user-name-handle
        display: inline-flex
        flex-wrap: wrap
        position: relative
        top: 30px

    div.u-user-name-handle a:first-child
        margin: 0 5px 0 10px
        text-decoration: none

    div.u-post-mentioned
        display: flex
        justify-content: flex-end
        flex-wrap: wrap
        width: 90%
        margin-left: auto
        margin-right: 20px

    div.u-post-mentioned a
        text-decoration: none

    div.u-post-date
        position: relative
        top: 30px

    button.u-like-react:hover
        color: $red-5!important

    button.u-comment:hover
        color: $primary!important

    button.u-repost:hover
        color: $green-5!important
</style>