<template>
    <div v-if="this.$props.posts.length > 0" class="u-all-post-box q-py-sm q-mb-md">
        <div class="u-audio-post-box q-ma-none q-pt-md" v-for="(post, index) in this.$props.posts" :key="post.key">
            <div class="q-pa-sm q-mb-none row">
                <q-avatar class="q-ml-md q-mt-md">
                    <img :src="post.data.user.avatar">
                </q-avatar>
                <div class="u-user-name-handle col-7">
                    <router-link class="text-subtitle2 text-bold text-primary" :to="'/profile?u='+post.data.user.handle.substring(1)" v-slot="{navigate}">
                        <span class="q-mr-xs q-ml-sm cursor-pointer" @click="navigate">{{post.data.user.name}}</span>
                    </router-link>
                    <div class="text-subtitle2 text-grey-8 text-weight-regular">{{post.data.user.handle}}</div>
                    <div class="block u-post-viewer-wrap text-primary text-weight-regular text-caption">
                        <q-btn flat rounded size=".65rem" no-caps color="primary" :icon="post.data.viewer.icon" :label="post.data.viewer.text+' can view'" />
                    </div>
                </div>
                <div class="text-subtitle2 text-grey col-3 u-post-date text-right">
                    {{ post.data.date | niceDate }}
                </div>
                <!-- <div class="col-2">
                    <q-icon :name="post.viewer.icon"></q-icon>
                    {{post.viewer.text+' can reply'}}
                </div> -->
            </div>
            <p class="u-post-caption text-grey-8" v-show="post.data.postCaption">{{post.data.postCaption}}</p>
            <AudioPlayer
                :uniqueID="'userIndex'+index"
                :AudioSRC="getAudioSource(post.data.audioBLOB)"
                :maxLength="post.data.audioLength"
                :closeBTN="false"
            />
            <div class="u-post-mentioned">
                <q-chip class="q-mt-md q-mb-none" v-for="(user, index) in post.data.mentionedUsers" :key="index">
                    <div class="cursor-pointer" @click="changeProfilePage('profile?u=',user.handle.substring(1))">
                        <q-avatar>
                            <img :src="user.avatar">
                        </q-avatar>
                        {{user.handle}}
                    </div>
                </q-chip>
            </div>
            <div class="u-post-action q-mt-xl">
                <q-btn-group flat spread>
                    <q-btn
                        @click="toggleLikes(post.key,post.data.likes)"
                        class="q-py-sm u-like-react"
                        :color="checkIfUSerLikeThePost(post.key) ? 'amber-13' : 'grey-5'"                        
                        :label="post.data.likes.length > 0 ? post.data.likes.length : ''"
                        flat
                    >
                        <template slot="default">
                            <img :class="checkIfUSerLikeThePost(post.key) ? 'u-dug-it' : 'dig-it'" src="icons/dig_it.png" style="height: 35px;" class="q-mx-sm" />                       
                            <span class="u-dig-it-tooltip">{{checkIfUSerLikeThePost(post.key) ? 'Undig?' : 'Dig it!'}}</span></template>
                    </q-btn>
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
            :thickness="5"
            color="grey-3"
            size="2.5rem"
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
        },
        changeProfilePage(link,handle) {
            this.$router.push('profile?u='+handle)
            this.$emit('changeProfilePage',link+handle)
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

    p.u-post-caption
        width: 90%
        margin-left: auto
        margin-right: 20px
        margin-top: 10px
        padding: 5px

    div.u-audio-post-box div.u-audio-player
        width: 90%
        margin-left: auto
        margin-top: 15px

    div.u-user-name-handle
        display: inline-flex
        flex-wrap: wrap
        position: relative
        top: 25px

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
        top: 25px

    button.u-like-react:hover
        color: $amber!important

    button.u-comment:hover
        color: $primary!important

    button.u-repost:hover
        color: $green-5!important

    img.dig-it
        opacity: .4

    button.u-like-react:hover img.dig-it
        opacity: 1

    img.u-dug-it
        opacity: 1

    span.u-dig-it-tooltip
        font-size: 8px!important
        font-weight: 700
        margin-top: -20px
        margin-left: 90px
        background-color: $grey-9!important
        padding: 0px 8px
        border-radius: 10px
        color: white
        visibility: hidden
        position: absolute
        text-transform: capitalize

    button.u-like-react:hover span.u-dig-it-tooltip
        visibility: visible

    div.u-post-viewer-wrap
        position: absolute
        top: 25px
        left: 10px
</style>