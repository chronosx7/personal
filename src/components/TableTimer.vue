<template>
    <div tabindex="1"
        @keyup.f1="add_record" 
        @keyup.esc="confirm_clear_records" 
        class="h-100"
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

        <v-container class="h-100">
            <v-row fill-height>
                <v-col cols="12">
                    <div class="centered-container h-100">
                        <div v-if="current_tab == Tabs.timer" class="h-100">

                            <v-dialog v-model="dialog" max-width="290" 
                            >
                                <v-card>
                                    <v-card-text>
                                        Are you sure you want to clear the table?
                                    </v-card-text>
                                    <v-card-actions>
                                        <v-spacer></v-spacer>
                                        <v-btn @click.stop="confirm_clear_records">Cancel</v-btn>
                                        <v-btn id="accept_clear_btn" 
                                        @click="clear_records">Clear</v-btn>
                                    </v-card-actions>
                                </v-card>
                            </v-dialog>

                            <div  class="stopwatch">
                                <Stopwatch ref="stopwatch"></Stopwatch>
                                <v-btn-group variant="outlined" style="margin-bottom: 5px;">
                                    <v-btn @click="add_record">Log Time(F1)</v-btn>
                                    <v-btn @click="copy_record">Copy</v-btn>
                                    <v-btn @click="confirm_clear_records">Clear(ESC)</v-btn>
                                </v-btn-group>
                            </div>
                            <div class="h-100" ref="table_container" id="events">
                                <EventsList :source-list="records" :headers="headers" v-on:row-removed="check_rows" :height="table_size"></EventsList>
                            </div>

                        </div>

                        <div v-if="current_tab == Tabs.history">
                            Record History
                        </div>

                        <div v-if="current_tab == Tabs.instructions">
                            <instructions />
                        </div>
                    </div>
                </v-col>
            </v-row>
        </v-container>
    </div>
</template>
<script setup lang="ts">
    import EventsList from './EventsList.vue'
    import Stopwatch, { StopwatchAPI } from './Stopwatch.vue';
    import { ref, watch, onMounted, nextTick } from 'vue'
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
    let dialog = ref(false)
    let current_focus:Element|null

    // Event table-related attributes
    let table_container = ref(null)
    let table_size = ref("300px")

    const handle_resize = () => {
        if (table_container.value) {
            const stopwatch_height = (document.querySelector(".stopwatch") as HTMLElement).offsetHeight
            const diff = window.innerHeight - stopwatch_height - 55 + "px"
            table_size.value = diff
        }
    }

    const stopwatch = ref(null)
    let stopwatch_ref: StopwatchAPI
    const records = ref([] as Record[])

    /**
     * Lifecycle hooks
     */
    onMounted(function(){
        stopwatch_ref = stopwatch.value as unknown as StopwatchAPI

        window.addEventListener('resize', handle_resize)
        handle_resize()
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
        nextTick(() => {
            const cells = document.querySelectorAll('#events tr td input')
            const field = cells.item(cells.length-1) as HTMLElement
            field.focus()
        })
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

    // watch(dialog, async (oldVal, newVal) => {
    //     if(newVal === true){
    //         console.log(`Focusing previous elem ${current_focus}`)
    //         nextTick(() => {
    //             (current_focus as HTMLElement).focus()
    //             current_focus = null
    //         })
    //     }
    // })
    function confirm_clear_records(){
        current_focus = dialog.value? current_focus: document.activeElement
        console.log(`current_focus: ${current_focus}`)
        dialog.value = !dialog.value
        nextTick(() => {
            if(dialog.value === true){
                console.log("Open dialog. Selecting clear button");
                (document.querySelector("#accept_clear_btn") as HTMLElement).focus()
            }
            if(dialog.value === false && current_focus !== null){
                console.log("Close dialog. Re-select element");
                (current_focus as HTMLElement).focus()
                current_focus
            }
        })
    }

    function clear_records(){
        records.value = []
        stopwatch_ref.reset_counter()
        dialog.value = false
    }

    function choose_tab(label:Tabs){
        current_tab.value = label
        console.log(`Selected tab: ${label}`)
    }

</script>
<style>

.centered-container{
    padding: 10px;
    max-width: 1600px;
    margin: 0 auto;
}
.fixed{
    position: fixed;
    height: 250px;
    border: solid 1px;
    width: 100%;
    top: 10px;
}
.offset-body{
    position: relative;
    margin: 200px auto 0 auto;
    width: 100%;
}
.bordered{
    border: solid 1px;
}
.red-border{
    border-color: red;
}
.green-border{
    border-color: green;
}
.h-100{
    display: flex;
    flex-direction: column;
    height: 100%;
}
.h-20{
    height: 20%;
}
.h-80{
    height: 80%;
}
</style>
