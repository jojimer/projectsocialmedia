<template>
    <q-page :class="transition ? 'fade-enter' : 'fade-enter-active'">
        <q-card class="my-card" square flat>
        <q-img
            src="https://cdn.quasar.dev/img/parallax1.jpg"
            height="11rem"
            no-default-spinner            
        />

        <q-card-section class="u-profile-avatar">
            <q-avatar size="8rem">
                <q-icon
                    v-if="user.avatar === ''"
                    style="background-color:#f1f1f1;padding:1.8rem 2.5rem 2.5rem;"
                    name="far fa-user"
                    size="4rem"
                />
                <q-img
                    v-else
                    height="8rem"
                    :src="user.avatar"
                    class="u-profile-picture"
                />
            </q-avatar>
        </q-card-section>
        <q-card-section v-if="userIsViewing" class="u-edit-profile">
            <div class="q-px-md q-gutter-sm text-right ">
                <q-btn rounded unelevated outline no-wrap color="primary" no-caps label="Edit profile" />
            </div>
        </q-card-section>
        <q-card-section v-else class="u-profile-action">
            <div class="q-px-md q-gutter-sm text-right ">
                <q-btn round outline color="primary" icon="fas fa-ellipsis-h" />
                <q-btn round unelevated color="primary" icon="fas fa-bell" />
                <q-btn rounded unelevated no-wrap color="primary" no-caps label="Listening" />
            </div>
        </q-card-section>
        <q-card-section class="u-profile-name-handle">
            <div class="q-px-md q-gutter-sm text-left  ">
                <p class="q-ma-none text-weight-bolder text-h6">{{user.name}}</p>
                <p class="text-subtitle2 q-ma-none text-grey-8 text-weight-regular" style="margin-top:-.4rem">{{user.handle}}</p>   
                <p class="text-grey-7 text-weight-regular">Lorem ipsum, dolor sit amet consectetur adipisicing elit. Quod laborum natus eaque minima sed repellendus voluptas repellat, maxime fuga veniam.</p>             
            </div>
            <div>
                <div class="text-subtitle2">
                    <span class="q-ml-md text-weight-bolder">0</span>
                    <span class="q-ml-xs text-weight-regular text-grey-7">Listening</span>
                    <span class="q-ml-md text-weight-bolder">0</span>
                    <span class="q-ml-xs text-weight-regular text-grey-7">Listeners</span>
                </div>
            </div>
        </q-card-section>
        <q-card-section class="q-px-none">
                <q-tabs
                    v-model="tab"
                    dense
                    class="text-grey"
                    active-color="primary"
                    indicator-color="primary"
                    align="justify"
                >
                    <q-tab class="q-py-sm" name="posts" label="Posts" />
                    <q-tab class="q-py-sm" name="replies" label="Post & Replies" />
                    <q-tab class="q-py-sm" name="likes" label="Likes" />
                    <q-tab class="q-py-sm" name="friends" label="Friends" />
                </q-tabs>
                <q-separator />
                <div class="q-gutter-y-xs">
                    <q-tab-panels v-model="tab" :animated="false">
                        <q-tab-panel class="q-pa-none" name="posts">
                            <AudioPost
                                :posts="userPosts"
                                :users="dummyUser"
                            />
                        </q-tab-panel>

                        <q-tab-panel name="replies">
                            <AudioPost
                                :posts="userLikes"
                                :users="dummyUser"
                            />
                        </q-tab-panel>

                        <q-tab-panel name="likes">
                            <AudioPost
                                :posts="userLikes"
                                :users="dummyUser"
                            />
                        </q-tab-panel>

                        <q-tab-panel name="friends">
                            <div class="text-center text-grey-5 q-py-lg q-mb-md"><h6 class="text-weight-regular">No friends available!</h6></div>
                        </q-tab-panel>
                    </q-tab-panels>
                </div>
        </q-card-section>
        </q-card>
    </q-page>
</template>

<script>
import AudioPost from '../components/AudioPost'
export default {
    data() {
        return {
            tab: 'posts',
            userIsViewing: false, 
            db: new this.$localbase('db'),
            user: {
                avatar: '',
                handle: '',
                name: '',
                postIDs: []
            },
            userPosts: [],
            userLikes: [],
            transition: false
        }
    },
    props: {
        currentUser: Object,
        dummyUser: Array,
        link: String
    },
    components: {
        AudioPost
    },
    methods: {
        checkIfUserIsViewing(){
            let handle = '@'+this.$route.query.u
            this.userIsViewing = handle === this.$props.currentUser.handle
            if(this.userIsViewing){
                this.user = this.$props.currentUser
            }else{
                let user = this.$props.dummyUser.filter(user => { if(user.handle === handle) return user})
                this.user = user[0]
            }
            setTimeout(() => {
                this.transition = false
            },300)
        },
        async setUserPost() {
            let userPosts = this.user.postIDs.reverse()
            let posts = []
            userPosts.map(postID => {
                this.db.collection('audiopost').doc(postID).get()
                    .then(result => {
                        posts.push(result)
                    }, (err) => {
                        console.log(err)
                    })
            })
            this.userPosts = posts
        }
    },
    created() {
        
    },
    mounted() {
        this.$emit('changeLink')
        this.db.config.debug = false
        this.setUserPost()
    },
    watch: {
        link() {            
            this.transition = true
            this.checkIfUserIsViewing()
            this.setUserPost()
        },
        dummyUser() {
            this.transition = true
            this.checkIfUserIsViewing()
            this.setUserPost()
        }
    }
}
</script>

<style scoped lang="sass">
.u-profile-avatar, .u-profile-action, .u-edit-profile
    margin-top: -5rem
</style>