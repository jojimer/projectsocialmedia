<template>
<div class="u-message-control q-px-md q-py-sm">
    <p class="q-mb-xs text-subtitle1">Say something...</p>
    <div>
        <input id="fileUpload" :value="sound.file" type="file" accept="audio/*" hidden>
        <q-icon @click="$emit('chooseFiles')" size="md" name="fas fa-file-upload" class="cursor-pointer text-primary q-pr-sm" />
        <q-icon
            v-show="microphone"
            name="fas fa-circle"
            @click="$emit('captureAudio')"
            class="cursor-pointer q-pr-sm"
            color="negative"
            size="md"
        />
    </div>
</div>
</template>

<script>
export default{
    props: {
        sound: Object,
        microphone: Boolean,
    }
}
</script>

<style scoped lang="sass">
    div.u-message-control
        border: 1px #5555 solid
        border-radius: 5px
</style>