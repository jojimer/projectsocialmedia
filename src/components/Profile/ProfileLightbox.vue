<template>
    <div>
        <q-dialog v-model="fixed" @before-hide="$emit('hideLightBox',false)" transition-show="fade">
        <q-card class="u-editprofile-card">
            <q-toolbar class="col-12 justify-between q-pr-md">                
                <div class="col">
                    <q-btn color="primary" flat round dense icon="close" style="margin-top: -10px;" v-close-popup />
                    <span class="text-h6 text-weight-bolder q-pt-lg q-mx-md">Edit profile</span>
                </div>
                <q-btn @click="updateProfileInfo" rounded label="Save" no-caps color="primary" :loading="false" />
            </q-toolbar>

            <q-separator />
            <div class="q-mt-xl scroll-y u-editor-wrap">
            <q-card-section class="q-pa-none" :class="cover === '' ? 'bg-grey' : ''">
                <q-img
                    v-if="cover !== ''"
                    :src="cover"
                    height="11rem"
                    width="30rem"
                />
                <q-img
                    v-else
                    height="11rem"
                    width="30rem"
                    no-default-spinner     
                />
                <q-icon size="1rem" class="u-upload-trigger" name="fas fa-camera" @click="chooseCover" />
                <input id="fileCover" type="file" class="hidden">
            </q-card-section>
            <q-card-section class="u-profile-avatar scroll">
                <q-avatar size="6rem">
                    <input id="filePP" type="file" class="hidden">
                    <q-icon size="1rem" class="u-upload-trigger" name="fas fa-camera" @click="chooseProfile" />
                    <q-icon
                        v-if="profile === ''"
                        style="background-color:#f1f1f1;padding:1.5rem 2.5rem 2.5rem;"
                        name="far fa-user"
                        size="3rem"
                    />
                    <q-img
                        v-else
                        height="6rem"
                        :src="profile"
                        class="u-profile-picture"
                    />
                </q-avatar>
            </q-card-section>
            <q-card-section class="u-inputfield-wrap q-gutter-xs">
                <q-input class="u-input-name" outlined v-model="name" label="Name" maxlength="50" counter />
                <q-input class="u-input-bio" outlined v-model="bio" type="textarea" label="Bio" maxlength="160" counter />
                <q-input class="u-input-location q-mt-md" outlined v-model="location" label="Location" maxlength="30" counter />
                <q-input class="u-input-website" outlined v-model="website" label="Website" maxlength="100" counter />
                <q-input outlined v-model="date" :rules="['MMMM Do, YYYY']" label="Birthday">
                <template v-slot:append>
                    <q-icon name="event" class="cursor-pointer">
                    <q-popup-proxy class="u-qdate" ref="qDateProxy" transition-show="scale" transition-hide="scale">
                        <q-date v-model="date" landscape mask="MMMM Do, YYYY" default-year-month="2000/01/01">
                        <div class="row items-center justify-end">
                            <q-btn v-close-popup label="Close" color="primary" flat />
                        </div>
                        </q-date>
                    </q-popup-proxy>
                    </q-icon>
                </template>
                </q-input>
            </q-card-section>
            </div>
        </q-card>
        </q-dialog>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                fixed: false,
                userInfo: {},
                date: this.$props.user.birthday,
                cover: this.$props.user.cover,
                profile: this.$props.user.avatar,
                name: this.$props.user.name,
                location: this.$props.user.location,
                bio: this.$props.user.bio,
                website: this.$props.user.website,
            }
        },
        props: {
            show: Boolean,
            user: Object
        },
        methods: {
            updateProfileInfo() {
                this.setUserInfo()
                this.$emit('saveProfileInfo',this.userInfo)
            },
            setUserInfo() {
                this.userInfo.avatar = this.profile
                this.userInfo.name = this.name
                this.userInfo.birthday = this.date
                this.userInfo.location = this.location
                this.userInfo.bio = this.bio
                this.userInfo.website = this.website
                this.userInfo.cover = this.cover
            },
            getImageBlob(input,place) {
                if (input) {
                    let reader = new FileReader()
                    reader.readAsDataURL(input)
                    reader.onload = (e) => {
                        // browser completed reading file - display it
                        if(place === 'cover'){
                            this.cover =  e.target.result
                        }else{
                            this.profile = e.target.result
                        }
                    }                    
                }
            },
            chooseCover() {
                let coverInput = document.querySelector("input#fileCover")
                coverInput.click()
                coverInput.addEventListener('change', (e) => {
                    this.getImageBlob(coverInput.files[0],'cover')
                },false)
            },
            chooseProfile() {
                let profileInput = document.querySelector("input#filePP")
                profileInput.click()
                profileInput.addEventListener('change', (e) => {
                    this.getImageBlob(profileInput.files[0],'profile')
                },false)
            },
        },
        watch: {
            show(val){
                this.fixed = val
            }
        }
    }
</script>
<style lang="sass" scoped>
.u-editprofile-card
    border-radius: 1rem,
    overflow-y: hidden

.u-editprofile-card .q-toolbar
    position: absolute
    z-index: 1
    background-color: white

.u-upload-trigger
    position: absolute
    top: 1.1rem
    right: 1.2rem
    z-index: 1
    color: #f1f1f1
    cursor: pointer

.u-editor-wrap
    max-height: 75vh

.u-profile-avatar, .u-profile-action, .u-edit-profile
    margin-top: -4rem

.u-inputfield-wrap
    margin-top: -1.5rem

div.q-menu.q-position-engine
    top: 18rem
    left: unset
    right: 1rem
</style>