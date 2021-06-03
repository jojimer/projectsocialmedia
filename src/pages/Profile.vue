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
                <q-btn
                    v-show="notificationBTN"
                    @click="toggleNotification(user.id)"
                    color="primary"
                    :icon="notified ? 'notifications_active':'notification_add'"
                    :outline="!notified"
                    :loading="notificationBTNLoading"
                    unelevated
                    round
                />
                <div class="u-btn-friend-request-wrap">
                    <q-btn
                        @click="toggleFriendButton(user.id,request)"
                        @mouseenter="friendRequestLabel === 'Friends' ? changeFriendsButtonLabel('enter') : null"
                        @mouseleave="friendRequestLabel === 'Unfriend' ? changeFriendsButtonLabel('leave') : null"
                        :color="friendRequestLabel === 'Unfriend' ? 'negative' : 'primary'"
                        no-caps
                        no-wrap
                        rounded
                        unelevated
                        :loading="friendbtnLoading"
                    >
                        <template slot="default">
                            <q-icon :name="friendRequestIcon" />
                            <span class="block q-ml-sm">{{friendRequestLabel}}</span>
                        </template>
                        <template slot="loading">
                            <q-spinner color="white" size="1.5rem" :thickness="5" />
                            <span class="block q-ml-sm">{{friendRequestLabel}}</span>
                        </template>
                    </q-btn>
                    <div class="u-btn-respond-wrap" v-show="toggleRespond">
                        <q-btn color="primary" flat align="left" label="accept" @click="requestRespond(user.id,'accept')" />
                        <q-btn color="primary" flat align="left" label="decline" @click="requestRespond(user.id,'decline')" />
                    </div>
                </div>
                <q-btn
                    @click="toggleFollow(user.id,user.followers,user.totalFollowers)"
                    color="primary"
                    no-caps
                    no-wrap
                    :outline="!following"
                    rounded
                    :unelevated="following"
                    :loading="followBTNLoading"
                    @mouseenter="followLabel ='Unlisten'"
                    @mouseleave="followLabel = 'Listening'"
                >
                    <template slot="default">
                        <span class="block" style="padding:4px 16px 3px;">{{!following ? 'Listen' : followLabel}}</span>
                    </template>
                    <template slot="loading">
                        <q-spinner :color="!following ? 'primary' : 'white'" size="1.4rem" :thickness="5" />
                        <span class="q-ml-sm">{{!following ? 'Listen' : followLabel}}</span>
                    </template>
                </q-btn>
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
                    <span class="q-ml-md text-weight-bolder">{{user.totalFriends}}</span>
                    <span class="q-ml-xs text-weight-regular text-grey-7">{{user.totalFriends > 1 ? 'Friends' : 'Friend'}}</span>
                    <span class="q-ml-md text-weight-bolder">{{user.totalFollowing}}</span>
                    <span class="q-ml-xs text-weight-regular text-grey-7">Listening</span>
                    <span class="q-ml-md text-weight-bolder">{{user.totalFollowers}}</span>
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
                    <q-tab v-show="user.totalFriends > 0" class="q-py-sm" name="friends" label="Friends" />
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
                                @changeProfilePage="this.$emit('changeProfilePage')"
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

                        <q-tab-panel name="friends" class="q-pa-none">
                            <div class="q-py-sm">
                                <Friendlist v-if="user.totalFriends > 0" :friends="user.friends" @changeProfilePage="(link) => this.$emit('changeProfilePage', link)" />
                                <h6 class="text-weight-regular text-center text-grey-5 q-py-sm q-mb-md" v-else>No friends available!</h6>
                            </div>
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
import Friendlist from '../components/Friends/FriendList'
export default {
    data() {
        return {
            tab: 'posts',
            userIsViewing: false, 
            db: new this.$localbase('db'),
            dbName: this.$databaseName,
            notificationBTN: false,
            user: {
                
            },
            following: false,
            followLabel: 'Listening',
            followBTNLoading: false,
            request: 'respond',
            friendRequestLabel: 'Add friend',
            friendRequestIcon: 'fas fa-user-plus',
            friendbtnLoading: false,
            toggleRespond: false,
            userPosts: [],
            userReplies: [],
            userFriends: [],
            userLikes: [],
            notified: false,
            notificationBTNLoading: false,
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
        ProfileLightbox,
        Friendlist
    },
    methods: {
        checkIfUserIsViewing(){
            let handle = '@'+this.$route.query.u
            this.tab = 'posts'
            this.userIsViewing = handle === this.$props.currentUser.handle
            if(this.userIsViewing){
                this.user = this.$props.currentUser
            }else{
                let user = this.$props.dummyUser.filter(user => { if(user.handle === handle) return user})
                this.user = user[0]
                this.checkIfProfileIsFriend(this.user.id)
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
        },
        async toggleFriendButton(id,request) {
            if(request !== 'respond') this.friendbtnLoading = true
            let userID = this.$props.currentUser.id
            let userFriends = this.$props.currentUser.friends
            let profileFriendRequests = this.user.pendingRequests
            let userPendingRequests = this.$props.currentUser.pendingRequests
            
            if(request !== 'unfriend' && !profileFriendRequests.includes(userID) && !userPendingRequests.includes(id)){
                userPendingRequests.push(id)
                profileFriendRequests.push(userID)
                await this.db.collection(this.dbName.users).doc('user-key'+userID).update({pendingRequests: userPendingRequests})
                await this.db.collection(this.dbName.users).doc('user-key'+id).update({friendRequests: profileFriendRequests})
            }else if(request.length > 0 && request !== 'add' && request !== 'respond'){
                let userObj
                let profileObj
                let userIndex
                let profileIndex                
                switch(request) {
                    case 'cancel':
                        userObj = {pendingRequests: userPendingRequests}
                        userIndex = userObj.pendingRequests.indexOf(id)
                        if (userIndex > -1) {
                            userObj.pendingRequests.splice(userIndex, 1)
                        }
                        profileObj = {friendRequests: this.user.friendRequests}
                        profileIndex = profileObj.friendRequests.indexOf(userID)
                        if (profileIndex > -1) {
                            profileObj.friendRequests.splice(profileIndex, 1)
                        }
                    break                 
                    case 'unfriend':
                        let userTotalFriends = this.$props.currentUser.totalFriends
                        let profileTotalFriends = this.user.totalFriends
                            userTotalFriends =  userTotalFriends > 0 ? userTotalFriends-1 : 0                    
                            profileTotalFriends = profileTotalFriends > 0 ? profileTotalFriends-1 : 0
                        userObj = {friends: userFriends, totalFriends: userTotalFriends}
                        userIndex = userObj.friends.map((friend) => friend.id).indexOf(id)
                        if (userIndex > -1) {
                            userObj.friends.splice(userIndex, 1)
                        }
                        profileObj = {friends: this.user.friends, totalFriends: profileTotalFriends}
                        profileIndex = profileObj.friends.map((friend) => friend.id).indexOf(userID)
                        if (profileIndex > -1) {
                            profileObj.friends.splice(profileIndex, 1)
                        }
                    break
                }
                
                if (userIndex > -1) {                    
                    console.log(request,profileIndex,userID)
                    //Remove from user data                    
                    await this.db.collection(this.dbName.users).doc('user-key'+userID).update(userObj)
                    await this.db.collection(this.dbName.users).doc('user-key'+id).update(profileObj)
                }
            }else{
                this.toggleRespond = true
            }

            setTimeout(() => {
                this.friendbtnLoading = false
                this.checkIfProfileIsFriend(id)
            },1500)
        },
        async requestRespond(id,type){
            this.friendbtnLoading = true
            let userID = this.$props.currentUser.id
            let userFriendRequests = this.$props.currentUser.friendRequests
            let userFriends = this.$props.currentUser.friends
            let userTotalFriends = this.$props.currentUser.totalFriends
            let profileFriends = this.user.friends
            let profilePendingRequest = this.user.pendingRequests
            let profileTotalFriends = this.user.totalFriends
            let userIndex
            let profileIndex

            let userObj = {friendRequests: userFriendRequests}
            userIndex = userObj.friendRequests.indexOf(id)
            if (userIndex > -1) {
                userObj.friendRequests.splice(userIndex, 1)
            }
            
            let profileObj = {pendingRequests: profilePendingRequest}
            profileIndex = profileObj.pendingRequests.indexOf(userID)
            if (profileIndex > -1) {
                profileObj.pendingRequests.splice(profileIndex, 1)
            }
            if(type == 'accept'){
                //save friends
                userFriends.push({id:id,name:this.user.name})
                userFriends.sort((a, b) => (a.name > b.name) ? 1 : -1)
                userTotalFriends = userTotalFriends+1
                await this.db.collection(this.dbName.users).doc('user-key'+userID).update({friends: userFriends, totalFriends: userTotalFriends})
                profileFriends.push({id:userID,name:this.$props.currentUser.name})
                profileFriends.sort((a, b) => (a.name > b.name) ? 1 : -1)
                profileTotalFriends = profileTotalFriends+1
                await this.db.collection(this.dbName.users).doc('user-key'+id).update({friends: profileFriends, totalFriends: profileTotalFriends})
                this.user.totalFriends = this.user.totalFriends+1
            }else{
                this.user.totalFriends = this.user.totalFriends-1
            }
            
            //Remove from user data                    
            await this.db.collection(this.dbName.users).doc('user-key'+userID).update(userObj)
            await this.db.collection(this.dbName.users).doc('user-key'+id).update(profileObj)
            this.toggleRespond = false
            
            setTimeout(() => {
                this.friendbtnLoading = false
                this.checkIfProfileIsFriend(id) 
            },2500)
        },
        checkIfProfileIsFriend(id){
            let userID = this.$props.currentUser.id
            let userFriends = this.$props.currentUser.friends
            let havePendingRequest = this.$props.currentUser.pendingRequests
            let profileHaveRequest = this.user.pendingRequests
            let checkIndex = userFriends.map((friend) => friend.id).indexOf(id)
            let isFriends = (checkIndex > -1) ? true : false

            if(this.request !== 'cancel'){
                if(isFriends){
                    this.notificationBTN = true
                    this.request = 'unfriend'
                    this.friendRequestLabel = 'Friends'
                    this.friendRequestIcon = 'fas fa-user-friends'
                }else if((havePendingRequest.indexOf(id) > -1) ? true : false){
                    this.request = 'cancel'
                    this.friendRequestLabel = 'Cancel Request'
                    this.friendRequestIcon = 'fas fa-user-times'
                }else if((profileHaveRequest.indexOf(userID) > -1) ? true : false){
                    this.request = 'respond'
                    this.friendRequestLabel = 'Respond'
                    this.friendRequestIcon = 'fas fa-user-check'
                }else{
                    this.request = 'add'
                    this.friendRequestLabel = 'Add Friend'
                    this.friendRequestIcon = 'fas fa-user-plus'
                }
            }else{
                this.request = 'add'
                this.friendRequestLabel = 'Add Friend'
                this.friendRequestIcon = 'fas fa-user-plus'
            }

            this.$emit('updateUserInfo')
        },        
        changeFriendsButtonLabel(action) {
            if(this.friendRequestLabel === 'Unfriend' && action === 'leave'){
                this.friendRequestLabel = 'Friends'
                this.friendRequestIcon = 'fas fa-user-friends'
            }else{
                this.friendRequestLabel = 'Unfriend'
                this.friendRequestIcon = 'fas fa-user-times'
            }
        },
        async toggleFollow(id,followers,tnFollowers){
            this.followBTNLoading = true
            let userID = this.$props.currentUser.id
            let userFollowing = this.$props.currentUser.following
            let totalFollowers
            let totalFollowing
            
            if(!userFollowing.includes(id) && !followers.includes(userID)){                
                followers.push(userID)
                totalFollowers = tnFollowers+1
                userFollowing.push(id)
                totalFollowing = this.$props.currentUser.totalFollowing+1
                await this.db.collection(this.dbName.users).doc('user-key'+id).update({followers: followers,totalFollowers: totalFollowers})
                await this.db.collection(this.dbName.users).doc('user-key'+userID).update({following: userFollowing,totalFollowing: totalFollowing})
                this.followLabel = 'Listening'
            }else{                
                let followersIndex = followers.indexOf(userID)
                let followingIndex = userFollowing.indexOf(id)
                if (followersIndex > -1 && followingIndex > -1) {
                    //Remove followers from profile
                    followers.splice(followersIndex, 1)
                    totalFollowers = tnFollowers !== 0 ? tnFollowers-1 : 0
                    await this.db.collection(this.dbName.users).doc('user-key'+id).update({followers: followers,totalFollowers: totalFollowers})
                    //Remove following to user data
                    userFollowing.splice(followingIndex, 1)                    
                    totalFollowing = this.$props.currentUser.totalFollowing !== 0 ? this.$props.currentUser.totalFollowing-1 : 0
                    await this.db.collection(this.dbName.users).doc('user-key'+userID).update({following: userFollowing,totalFollowing: totalFollowing})
                }
            }
            setTimeout(() => {
                this.followBTNLoading = false
                this.checkIfUserFollowTheProfile()
                this.user.totalFollowers = totalFollowers
            },1500)
        },
        checkIfUserFollowTheProfile(){
            let id = this.user.id
            let userFollowing = this.$props.currentUser.following
            const following = userFollowing.indexOf(id)
            this.following = (following > -1) ? true : false
            this.notificationBTN = this.following ? true : false
        },
        async toggleNotification(id){
            this.notificationBTNLoading = true
            let userID = this.$props.currentUser.id
            let userNotice = this.$props.currentUser.usersNotify
            
            if(!userNotice.includes(id)){
                userNotice.push(id)
                await this.db.collection(this.dbName.users).doc('user-key'+userID).update({usersNotify: userNotice})
            }else{
                let noticeIndex = userNotice.indexOf(id)
                if (noticeIndex > -1) {
                    //Remove notification bell to user data
                    userNotice.splice(noticeIndex, 1)
                    await this.db.collection(this.dbName.users).doc('user-key'+userID).update({usersNotify: userNotice})
                }
            }
            setTimeout(() => {
                this.notificationBTNLoading = false
                this.checkIfUserNotification()
            },1500)
        },
        checkIfUserNotification(){
            let id = this.user.id
            let userNotice = this.$props.currentUser.usersNotify
            const notificationIndex = userNotice.indexOf(id)
            this.notified = (notificationIndex > -1) ? true : false
        },
        initProfilePage(){
            this.checkIfUserIsViewing()
            this.setUserPost()
            this.setUserLikes()  
            this.checkIfUserFollowTheProfile()
            this.checkIfUserNotification()
            this.transition = true
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
                this.initProfilePage()
            }
        },
        dummyUser() {
            this.initProfilePage()
        }
    }
}
</script>

<style scoped lang="sass">
.u-profile-avatar, .u-profile-action, .u-edit-profile
    margin-top: -5rem

.u-btn-friend-request-wrap 
    display: inline-flex
    position: relative
    vertical-align: middle
    height: 38px

.u-btn-respond-wrap
    position: absolute
    top: 2.5rem
    left: 0
    text-align: left
    width: 180px
    z-index: 1
    padding: 5px .4rem
    border-radius: 5px
    background-color: $grey-1  
    border: 1px solid $grey-4

.u-btn-respond-wrap button
    width: 100% 
</style>