<template>    
    <q-page :class="transition ? 'fade-enter' : 'fade-enter-active'">
        <ProfileLightbox 
        v-if="userIsViewing"
        :user="{
            avatar: user.avatar,
            name: user.name,
            cover: user.cover,
            bio: user.bio,
            location: user.location,
            website: user.website,
            birthday: user.birthday
        }" 
        :show="lightbox"        
        @saveProfileInfo="saveProfileInfo"
        @hideLightBox="(val) => lightbox = val "/>
        <q-card class="my-card" square flat>
        <q-img
        v-if="user.cover"
            :src="user.cover"
            height="11rem"
            no-default-spinner        
            class="thisHasContent"    
        />
        <q-img
            v-else
            height="11rem"
            no-default-spinner
            class="bg-grey"
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
                <q-btn @click="lightbox = true" rounded unelevated outline no-wrap color="primary" no-caps label="Edit profile" />
            </div>
        </q-card-section>
        <q-card-section v-else class="u-profile-action">
            <div class="q-px-md q-gutter-sm text-right ">
                <q-btn round outline color="primary" icon="fas fa-ellipsis-h" />
                <q-btn round unelevated color="primary" icon="fas fa-bell" />
                <q-btn rounded unelevated no-wrap color="primary" icon="fas fa-user-plus" no-caps label="Add friend" />
                <q-btn rounded unelevated no-wrap color="primary" no-caps label="Listening" />
            </div>
        </q-card-section>
        <q-card-section class="u-profile-name-handle">
            <div class="q-px-md q-gutter-sm text-left">
                <p class="q-ma-none text-weight-bolder text-h6">{{user.name}}</p>
                <p class="text-caption q-ma-none text-grey-7 text-weight-regular" style="margin-top:-.4rem">{{user.handle}}</p>   
                <p v-show="user.bio" class="text-grey-10 text-weight-regular">{{user.bio}}</p>      
                <div class="q-mb-sm text-grey-7 q-gutter-sm">
                    <span v-show="user.location" class="q-ml-none"><q-icon name="fas fa-map-marker-alt" /> {{user.location}}</span>
                    <span v-show="user.website"><q-icon name="fas fa-link" /> 
                        <a
                            style="text-decoration:none;"
                            :href="user.website"
                            class="q-px-sm text-primary"
                            target="_blank">{{user.website}}
                        </a>
                    </span>
                    <span v-show="user.birthday"><q-icon name="fas fa-gift" /> {{user.birthday}}</span>
                </div>       
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
                                :userID="currentUser.id"
                                :userLikes="currentUser.likes"
                            />
                        </q-tab-panel>

                        <q-tab-panel name="replies">
                            <AudioPost
                                :posts="userReplies"
                                :users="dummyUser"
                                :userID="currentUser.id"
                                :userLikes="currentUser.likes"
                            />
                        </q-tab-panel>

                        <q-tab-panel name="likes">
                            <AudioPost
                                :posts="userLikes"
                                :users="dummyUser"
                                :userID="currentUser.id"
                                :userLikes="currentUser.likes"
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
import ProfileLightbox from '../components/Profile/ProfileLightbox'
import AudioPost from '../components/AudioPost'
export default {
    data() {
        return {
            tab: 'posts',
            userIsViewing: false, 
            db: new this.$localbase('db'),
            dbName: this.$databaseName,
            user: {
                
            },
            userPosts: [],
            userReplies: [],
            userLikes: [],
            transition: false,
            lightbox: false
        }
    },
    props: {
        currentUser: Object,
        dummyUser: Array,
        link: String
    },
    components: {
        AudioPost,
        ProfileLightbox
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
            setTimeout(() => {
                let userPosts = this.user.postIDs.reverse()
                let posts = []
                userPosts.map(key => {
                    this.db.collection(this.dbName.audiopost).doc(key).get({ keys: true })
                        .then(result => {
                            let data = {
                                key: 'post-key'+result.postID,
                                data: result
                            }
                            posts.push(data)                            
                        }, (err) => {
                            console.log(err)
                        })
                })

                this.userPosts = posts
            },1500)
        },
        async setUserLikes() {
            setTimeout(() => {
                let userPosts = this.user.likes.reverse()
                let posts = []
                userPosts.map(postID => {
                    this.db.collection(this.dbName.audiopost).doc(postID).get({ keys: true })
                        .then(result => {
                            let data = {
                                key: 'post-key'+result.postID,
                                data: result
                            }
                            posts.push(data)
                        }, (err) => {
                            console.log(err)
                        })
                })

                this.userLikes = posts
            },1500)
        },
        async updatePostsUsersInfo(newData){
            let updatedUser = {
                id: newData.id,
                name: newData.name,
                handle: newData.handle,
                avatar: newData.avatar
            }

            let posts = newData.postIDs
            await posts.map(id => {
                this.db.collection(this.dbName.audiopost).doc(id).update({user: updatedUser})
            })
            
            setTimeout(() => {
                this.setUserPost()
                this.setUserLikes()
            },1500)
        },
        async saveProfileInfo(obj) {
            await this.db.collection(this.dbName.users).doc('user-key'+this.user.id).update(obj)
            let newData = await this.db.collection(this.dbName.users).doc('user-key'+this.user.id).get()
            await this.updatePostsUsersInfo(newData)
            this.user = newData
            this.$emit('updateUserInfo')
            this.lightbox = false
        }
    },
    created() {
        
    },
    mounted() {
        this.$emit('changeLink')
        this.db.config.debug = false
        this.setUserPost()
        this.setUserLikes()
    },
    watch: {
        link(val) {            
            if(val === 'profile' || this.$route.query.u){
                this.transition = true
                this.checkIfUserIsViewing()
                this.setUserPost()
                this.setUserLikes()
            }
        },
        dummyUser() {
            this.transition = true
            this.checkIfUserIsViewing()
            this.setUserPost()
            this.setUserLikes()
        }
    }
}
</script>

<style scoped lang="sass">
.u-profile-avatar, .u-profile-action, .u-edit-profile
    margin-top: -5rem
</style>