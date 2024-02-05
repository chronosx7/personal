<template>
    <div tabindex="1"
        @keyup.f1="add_record" 
        @keyup.esc="clear_records" 
    >
        <v-navigation-drawer
                v-model="drawer"
                :rail="rail"
                permanent
                absolute
            >
            <v-list >
                <v-list-item
                    prepend-icon="mdi-chevron-right"
                    @click.stop="rail = !rail">
                </v-list-item>
                <v-divider :thickness="2"></v-divider>

                <v-list-item prepend-icon="mdi-timer" title="Timer" value="timer"
                    @click="choose_tab(Tabs.timer)"></v-list-item>
                <!-- <v-list-item prepend-icon="mdi-history" title="History" value="history"
                    @click="choose_tab(Tabs.history)"></v-list-item> -->
                <v-list-item prepend-icon="mdi-book-open-page-variant" title="Instructions" value="instructions"
                    @click="choose_tab(Tabs.instructions)"></v-list-item>
            </v-list>
        </v-navigation-drawer>

        <v-main>
            <v-row>
                <v-col>
                    <div v-if="current_tab == Tabs.timer">
                        <Stopwatch ref="stopwatch"></Stopwatch>
                        <v-btn-group variant="outlined" style="margin-bottom: 5px;">
                            <v-btn @click="add_record">Log Time(F1)</v-btn>
                            <v-btn @click="copy_record">Copy</v-btn>
                            <v-btn @click="clear_records">Clear(ESC)</v-btn>
                        </v-btn-group>
                        <EventsList :source-list="records" :headers="headers" v-on:row-removed="check_rows"></EventsList>
                    </div>

                    <div v-if="current_tab == Tabs.history">
                        Record History
                    </div>

                    <div v-if="current_tab == Tabs.instructions">
                        <instructions />
                    </div>
                </v-col>
            </v-row>
        </v-main>
    </div>
</template>
<script setup lang="ts">
    import EventsList from './EventsList.vue'
    import Stopwatch, { StopwatchAPI } from './Stopwatch.vue';
    import { ref, onMounted } from 'vue'
    import Instructions from './Instructions.vue';

    enum Tabs {
        timer=0,
        history,
        instructions
    }
    const headers = [
        'Time',
        'Elapsed',
        "Description"
    ]
    type Record = {
        time: string;
        elapsed: string;
        description: string
    }
    let drawer = ref(true)
    let rail = ref(true)
    let current_tab = ref(Tabs.timer)

    const stopwatch = ref(null)
    let stopwatch_ref: StopwatchAPI
    const records = ref([] as Record[])

    /**
     * Lifecycle hooks
     */
    onMounted(function(){
        stopwatch_ref = stopwatch.value as unknown as StopwatchAPI
    })

    /**
     * Other functions
     */
    function add_record(){
        records.value.push({
            time: stopwatch_ref.get_current_time(),
            elapsed: stopwatch_ref.get_elapsed_time(),
            description: ''
        })
        stopwatch_ref.reset_counter()
        stopwatch_ref.start_counter()
    }

    function check_rows(){
        if(records.value.length == 0){
            stopwatch_ref.reset_counter()
        }
    }

    function copy_record(){
        console.log("Pressed F2")
        const record_text:string = records.value.reduce((total, current) => {
            return total += `${current.time}\t${current.elapsed}\t${current.description}\r\n`
        }, "")

        copy_text(record_text)
    }

    async function copy_text(text:string){
        try{
            await navigator.clipboard.writeText(text)
        }
        catch(err){
            console.error("Failed to copy records")
            console.error(err)
        }
    }

    function clear_records(){
        records.value = []
        stopwatch_ref.reset_counter()
    }

    function choose_tab(label:Tabs){
        current_tab.value = label
        console.log(`Selected tab: ${label}`)
    }


</script>