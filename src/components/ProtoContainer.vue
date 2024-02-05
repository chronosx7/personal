<template>
    <div>
        <div>
            <Stopwatch ref="stopwatch"></Stopwatch>
        </div>
        <div>
            <v-btn-group divided>
                <v-btn density="comfortable" @click="add_row">Add Row</v-btn>
                <span v-show="false">
                    <v-btn density="comfortable" @click="content_snapshot">Check Contents</v-btn>
                    <v-btn density="comfortable" @click="stopwatch_start">Start</v-btn>
                    <v-btn density="comfortable" @click="stopwatch_pause">Pause</v-btn>
                    <v-btn density="comfortable" @click="stopwatch_stop">Stop</v-btn>
                </span>
            </v-btn-group>
        </div>
        <div>
            <EventsList :source-list="lists" :headers="list_headers[current_list]"/>
        </div>
    </div>
</template>
<script setup lang="ts">
    import { ref, onMounted } from 'vue';
    import EventsList from './EventsList.vue';
    import Stopwatch, { StopwatchAPI } from './Stopwatch.vue';

    const stopwatch = ref(null)
    
    let stopwatch_ref: StopwatchAPI

    type Hero = {
        name: string;
        type: string;
        skill: string;
    }
    const list_heroes:Hero[] = [
        {
            name: 'Iskandar',
            type: 'Rider',
            skill: 'Aionian Hetairoi'
        },
        {
            name: 'Arturia Pendragon',
            type: 'Saber',
            skill: 'Excaliburn'
        },
        {
            name: 'Gilgamesh',
            type: 'Archer',
            skill: 'Gate of Babylon'
        }
    ]

    const current_list = ref()
    current_list.value = 0

    const lists = ref(list_heroes)
    const list_headers = [
        ['Name', 'Class', 'Noble Phantasm'],
        ['Name', 'Category'],
        ['Name', 'Role', 'Project', 'City'],
    ]

    onMounted(function(){
        stopwatch_ref = stopwatch.value as unknown as StopwatchAPI
    })
    function content_snapshot(){
        console.log(lists.value)
    }

    function add_row(){
        lists.value.push({
            name: stopwatch_ref.get_current_time(),
            type: stopwatch_ref.get_elapsed_time(),
            skill: 'Proto timestamp entry'
        })
        stopwatch_ref.reset_counter()
        stopwatch_ref.start_counter()
    }

    function stopwatch_start(){
        stopwatch_ref.start_counter()
    }
    function stopwatch_pause(){
        stopwatch_ref.pause_counter()
    }
    function stopwatch_stop(){
        stopwatch_ref.reset_counter()
    }

</script>
