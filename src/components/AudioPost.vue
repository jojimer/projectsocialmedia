<template>
    <div v-if="this.$props.posts.length > 0" class="u-all-post-box q-py-sm q-mb-md">
        <div class="u-audio-post-box q-ma-none q-pt-md" v-for="(post, index) in this.$props.posts" :key="post.user.name+'-'+post.postID">
            <div class="q-pa-sm q-mb-none row">
                <q-avatar class="q-ml-md q-mt-md">
                    <img :src="post.user.avatar">
                </q-avatar>
                <div class="u-user-name-handle col-7">
                    <router-link class="text-subtitle2 text-bold text-primary" :to="'/profile?u='+post.user.handle.substring(1)">{{post.user.name}}</router-link>
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
            <div class="u-post-action q-mt-lg">
                <q-btn-group flat spread>
                    <q-btn
                        class="q-py-sm u-like-react"
                        color="grey-5"
                        icon="far fa-heart"
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
            likeColor: []
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
        width: 85%
        margin-left: auto
        margin-right: 20px

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