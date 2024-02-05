<template>
    <p class="clock">
        {{current_time.hour}}:{{current_time.minute}}:{{current_time.second}}
        <span v-show="counting"> | {{elapsed_time.hour}}:{{elapsed_time.minute}}:{{elapsed_time.second}}</span>
    </p>
    <v-btn-group v-if="props.show_controls">
        <v-btn @click="start_counter">Start</v-btn>
        <v-btn @click="pause_counter">Pause</v-btn>
        <v-btn @click="reset_counter">Reset</v-btn>
    </v-btn-group>
</template>
<script setup lang="ts">
    import { ref } from 'vue';

    const props = defineProps({
        show_controls: {type: Boolean, required: false, default: false}
    })

    let counting = false
    let current_time = ref({hour: 0, minute: 0, second: 0})
    let elapsed_time = ref({hour: 0, minute: 0, second: 0})
    let counter_id = 0

    defineExpose({
        start_counter,
        pause_counter,
        reset_counter,
        get_current_time,
        get_elapsed_time
    })
    
    // This must match defineExpose for type validation
    export type StopwatchAPI = {
        start_counter(): void;
        pause_counter(): void;
        reset_counter(): void;
        get_current_time(): string;
        get_elapsed_time(): string;
    }
    
    window.setInterval(function(){
        const now = new Date()
        current_time.value.hour = now.getHours()
        current_time.value.minute = now.getMinutes()
        current_time.value.second = now.getSeconds()
    }, 1000)

    function start_counter(){
        if(counting == false){
            counter_id = window.setInterval(function(){
                elapsed_time.value.second += 1
                if(elapsed_time.value.second >= 60){
                    elapsed_time.value.second = 0
                    elapsed_time.value.minute += 1
                }
                if(elapsed_time.value.minute >= 60){
                    elapsed_time.value.minute = 0
                    elapsed_time.value.hour += 1
                }
            }, 1000)
            counting = true
        }
    }

    function pause_counter(){
        window.clearInterval(counter_id)
    }

    function reset_counter(){
        window.clearInterval(counter_id)
        elapsed_time.value.hour = 0
        elapsed_time.value.minute = 0
        elapsed_time.value.second = 0
        counting = false
    }

    function get_current_time(){
        return `${current_time.value.hour}:${current_time.value.minute}:${current_time.value.second}`
    }
    function get_elapsed_time(){
        return `${elapsed_time.value.hour}:${elapsed_time.value.minute}:${elapsed_time.value.second}`
    }
</script>
<style>
    .clock{
        font-size: 4em;
        margin: 1px;
    }
</style>