<template>
<div class="col">    
    <q-btn
        :loading="loading"
        :percentage="percentage"
        class="u-message-control q-px-md q-py-sm"
        dense
        unelevated
    >
        <q-icon name="fas fa-stop-circle" />
        <template v-slot:loading>
            <span>{{timer}}</span>
            <q-icon name="fas fa-stop-circle" @click="$emit('captureAudio')" />
        </template>
    </q-btn>
</div>
</template>

<script>
export default{
    data() {
        return {
            loading: true,
            percentage: 0,
            timer: '00:00',
            totalTime: 0
        }
    },
    props: {
        sound: Object,
        microphone: Boolean,
        currentTime: Number,
        limit: Number
    },
    methods: {
        updateTime(sec){
            this.timer = (sec <= 9) ? '0:0'+sec : 
            (sec <= 59) ? '0:'+sec : '1:00' 
            
            this.totalTime = sec
        }
    },
    created() {

    },
    watch: {
        currentTime(val,old){
            //Increment loading percentage
            this.percentage = this.percentage+.166666
            let newNumber = Math.ceil(val.toFixed(2))
            let oldNumber = Math.ceil(old.toFixed(2))
            if(newNumber > oldNumber){
                this.updateTime(newNumber)
            }            
        },
        totalTime(val){
            this.$emit('totalTime',val)
        }
    }
    
}
</script>

<style scoped lang="sass">
    button.u-message-control
        border-radius: 1rem!important
        width: 100%
        background-color: #17645D
        color: white    

</style>