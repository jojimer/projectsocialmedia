<template>
    <div class="u-all-post-box q-py-sm q-mb-md">
        <div class="u-audio-post-box q-ma-none q-py-md" v-for="(post, index) in this.$props.posts" :key="post.user.name+'-'+post.postID">
            <div class="q-pa-sm q-mb-none row">
                <q-avatar class="q-ml-md q-mt-md">
                    <img :src="post.user.avatar">
                </q-avatar>
                <div class="u-user-name-handle col-7">
                    <div class="text-subtitle2 text-bold text-primary">{{post.user.name}}</div>
                    <div class="text-subtitle2 text-grey-8 text-weight-regular">{{post.user.handle}}</div>
                </div>
                <div class="text-subtitle2 text-grey col-3 u-post-date text-right">
                    {{ post.date | niceDate }}
                </div>
                <!-- <div class="col-2">
                    <q-icon :name="post.viewer.icon"></q-icon>
                    {{post.viewer.text+' can reply'}}
                </div> -->
            </div>
            <AudioPlayer
                :uniqueID="'userIndex'+index"
                :AudioSRC="getAudioSource(post.audioBLOB)"
                :maxLength="post.audioLength"
                :closeBTN="false"
            />
            <div class="u-post-mentioned">
                <q-chip class="q-my-md" v-for="(user, index) in post.mentionedUsers" :key="index">
                    <q-avatar>
                        <img :src="user.avatar">
                    </q-avatar>
                    {{user.handle}}
                </q-chip>
            </div>
        </div>
    </div>
</template>

<script>
import AudioPlayer from './AudioPlayer'
import { date } from "quasar"

export default {
    data() {
        return {
            muted: true,
        }
    },
    props: {
        posts: Array,
        users: Array
    },
    components: {
        AudioPlayer
    },
    methods: {
        getAudioSource(blob){
            return URL.createObjectURL(blob)
        }
    },
    created() {
        
        // let newPost = []
        // this.posts = this.$props.preposts.map((post,index) => {
        //     let user = this.$props.users.filter(user => user.id == post.userID)
        //     post.mentionedUsers.map((id,index) => {
        //         let userMention = this.$props.users.filter(user => user.id == id)
        //         post.mentionedUsers[index] = userMention
        //     })
        //     post.user = user
        //     newPost.push(post)
        // })
        // this.posts = newPost
        //console.log(this.posts[0].mentionedUsers, this.posts[0].user[0].avatar)
    },
    filters: {
        niceDate(time) {
            return date.formatDate(time, "MMMM D, h:mmA ");
        },
    },
}
</script>
<style scoped lang="sass">
    div.u-all-post-box
        background-color: $grey-1

    div.u-audio-post-box
        height: 250px
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

    div.u-user-name-handle div:first-child
        margin: 0 5px 0 10px

    div.u-post-mentioned
        display: flex
        justify-content: flex-end
        width: 85%
        margin-left: auto
        margin-right: 20px

    div.u-post-date
        position: relative
        top: 30px
</style>