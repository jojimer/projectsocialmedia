<template>
    <div class="u-audio-player q-pa-sm q-ml-md q-mr-lg" :style="!closeBTN ? 'border-radius:1rem' : ''">
        <audio :id="'audioPlayer'+uniqueID" class="hidden" :src="AudioSRC" controls preload="auto"></audio>
        <div class="u-audio-player-wrap">
            <q-icon v-show="closeBTN" class="u-close-player" size="1.25rem" name="fas fa-times-circle" color="red-5" @click="$emit('close-player',false)" />
            <span class="q-pl-md q-py-sm u-timer">{{runningTime}}</span>
            <q-slider
                class="q-mx-md u-player-slider"
                v-model="currentTime"
                @change="val => {changeCurrentTime(val)}"
                :min="0"
                :max="audioDuration"
                :step="0.1"
                color="white"
                label
                label-color="grey-11"
                label-text-color="grey-7"
            />
            <span class="q-pr-sm q-py-sm u-timer">{{calculateTime(audioDuration)}}</span>
            <span class="u-volume-wrap q-mr-sm">
                <q-icon
                    :name="volume > 0 ? 'volume_up' : 'volume_off'"
                    @click="toggleVolume"
                    class="q-pa-sm"
                    color="white"
                    size="sm"
                />
                <q-slider
                    class="q-mx-xs"
                    v-model="volume"
                    @change="val => {volume = val}"
                    :min="0"
                    :max="1"
                    :step=".1"
                    color="white"
                />
            </span>
            <q-btn
                class="q-mx-sm u-play-button"
                padding="sm md"
                color="white"
                :icon="(playState !== 'play') ? 'fas fa-play' : 'fas fa-pause'"
                size="sm"
                flat
                dense
                @click="getAudioPlayer"
            />
        </div>
    </div>
</template>

<script>
    export default{
        data(){
            return {
                currentTime: 0,
                playState: 'pause',
                runningTime: '0:00',
                audioDuration: 0,
                player: '',
                volume: 1,
                previousVolume: 1
            }
        },
        props: {
            AudioSRC: String,
            maxLength: Number,
            closeBTN: Boolean,
            uniqueID: String
        },
        methods: {
            calculateTime(secs) {
                const minutes = Math.floor(secs / 60)
                const seconds = Math.floor(secs % 60)
                const returnedSeconds = seconds < 10 ? `0${seconds}` : `${seconds}`
                return `${minutes}:${returnedSeconds}`
            },
            getAudioPlayer() {
                this.player = document.querySelector('#audioPlayer'+this.$props.uniqueID)
                let duration = this.audioDuration

                let x = setInterval(() => {
                    if(duration > this.currentTime && this.playState == 'play'){
                        let n = this.currentTime+0.1
                        this.currentTime = Math.round(n * 10) / 10
                    }else{
                        clearInterval(x)
                        this.player.pause()
                        this.player.currentTime = 0
                        this.playState = 'pause'
                        if(duration <= this.currentTime){
                            this.currentTime = 0
                        }                        
                    }                        
                },100)
                

                if(this.playState !== 'play') {
                    this.player.volume = this.volume
                    this.player.play();
                    this.playState = 'play'
                } else {
                    this.player.pause();
                    this.playState = 'pause'
                }
            },
            updateTime(sec){
                this.runningTime = (sec <= 9) ? '0:0'+sec : 
                (sec <= 59) ? '0:'+sec : '1:00'
            },
            changeCurrentTime(time){
                this.currentTime = time
                this.player.currentTime = time
            },
            toggleVolume(){
                this.volume = (this.volume === 0) ? this.previousVolume : 0
            },
            getAudioDuration() {
                if(this.$props.maxLength === 0 && this.$props.AudioSRC !== ''){
                    setTimeout(() => {
                        let audio = document.querySelector('#audioPlayer'+this.$props.uniqueID);
                        this.audioDuration = Math.ceil(audio.duration.toFixed(2)) - 1
                    },1000)
                }else{
                    this.audioDuration = this.$props.maxLength
                }
            }
        },
        watch: {
            currentTime(val,old) {
                let newNumber = Math.ceil(val.toFixed(2))
                let oldNumber = Math.ceil(old.toFixed(2))
                if(newNumber > oldNumber && this.playState === 'play'){
                    this.updateTime(newNumber)
                }else{
                    this.updateTime(newNumber)
                }
            },
            volume(volume,previousVolume){
                if(volume === 0) this.previousVolume = previousVolume
                this.player.volume = volume
                
            },            
            audioDuration(newDuration){
                this.$emit('audioDuration',newDuration)
                //console.log(newDuration)
            }
        },
        mounted() {
            this.getAudioDuration()            
        }
    }
</script>

<style scoped lang="sass">
    div.u-audio-player
        width: 100%
        background-color: #17645D
        border-radius: 1rem 0 1rem 1rem
        position: relative

    div.u-audio-player-wrap
        display: flex
        justify-content: space-around

    .u-play-button:hover
        background-color: rgb(255 255 255 / 10%)

    div.u-audio-player-wrap span.u-timer
        color: white

    .u-close-player
        position: absolute
        right: -5.5px
        top: -5px
        cursor: pointer

    span.u-volume-wrap
        display: flex
        min-width: 120px
</style>