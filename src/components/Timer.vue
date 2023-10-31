<template>
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
                <v-divider :thickness="3"></v-divider>

                <v-list-item prepend-icon="mdi-timer" title="Timer" value="timer"
                    @click="choose_tab(Tabs.timer)"></v-list-item>
                <v-list-item prepend-icon="mdi-history" title="History" value="history"
                    @click="choose_tab(Tabs.history)"></v-list-item>
                <v-list-item prepend-icon="mdi-book-open-page-variant" title="Instructions" value="instructions"
                    @click="choose_tab(Tabs.instructions)"></v-list-item>
            </v-list>
        </v-navigation-drawer>

        <v-main>
            <v-row>
                <v-col>
                    <div v-if="current_tab == Tabs.timer">
                        <v-row>
                            <v-col>
                                <p class="clock">
                                    {{hour}}:{{minute}}:{{second}}
                                    <span v-show="counting"> | {{hours_passed}}:{{minutes_passed}}:{{seconds_passed}}</span>
                                </p>
                            </v-col>
                        </v-row>
                        <v-row>
                            <v-col>
                                <p>
                                    Log Time Difference: <input type="checkbox" v-model="log_difference"/>
                                </p>

                                <div class="my-2">
                                    <v-btn @click="log_time" rounded="lg">
                                        Log Time (F1)
                                    </v-btn>
                                    <v-btn @click="copy_timestamps" rounded="lg" class="mx-1">
                                        Copy All (F2)
                                    </v-btn>
                                    <v-btn @click="clear_timestamps" rounded="lg">
                                        Clear (Esc)
                                    </v-btn>
                                </div>

                                <v-textarea variant="outlined" id="timestamps"
                                    @keyup.f1.exact="log_time"
                                    @keyup.esc.exact="clear_timestamps" 
                                    @keyup.f2.exact="copy_timestamps"
                                    v-model="timestamps">
                                </v-textarea>
                            </v-col>
                        </v-row>
                    </div>
                    <div v-if="current_tab == Tabs.history">
                        <div class="my-2">
                            <v-btn rounded="lg" @click="clear_history">Clear History</v-btn>
                        </div>
                        <v-textarea variant="outlined" id="history" v-model="history"></v-textarea>
                    </div>
                    <div v-if="current_tab == Tabs.instructions">
                        <Instructions />
                    </div>
                </v-col>
            </v-row>
        </v-main>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import Instructions from './Instructions.vue'

enum Tabs {
    timer=0,
    history,
    instructions
}

export default defineComponent({
    data: function(){
        return{
            timestamps: '',
            hour: '',
            minute: '',
            second: '',
            counter_id: 0,
            counting: false,
            hours_passed: 0,
            minutes_passed: 0,
            seconds_passed: 0,
            last_hour: 0,
            last_minute: 0,
            last_second: 0,
            history: '',
            log_difference: false,
            show_instructions: false,
            instructions_label: 'Show Instructions',
            label_show: 'Show Instructions',
            label_hide: 'Hide Instructions',
            current_tab: 0,
            Tabs: Tabs,
            drawer: true,
            rail: true
        }
    },
    components: {
        Instructions: Instructions
    },
    mounted: function(){
        window.setInterval(function(self:any){
            let now = new Date()
            self.hour = now.getHours()
            self.minute = now.getMinutes()
            self.second = now.getSeconds()
        }, 100, this)
        this.select_timestamps_field()
        this.current_tab = Tabs.timer
    },
    methods:{
        choose_tab: function(label:Tabs){
            this.current_tab = label
        },
        log_time: function(){
            const curr_time = this.get_current_time()
            
            if(this.log_difference){
                if(this.timestamps !== ''){
                    this.timestamps += ` (Lasted ${this.get_passed_time()})\r\n`
                }
                this.timestamps += `${curr_time}`
            }
            else{
                this.timestamps += `${curr_time}\r\n`
            }

            this.activate_counter()
            this.select_timestamps_field()
        },
        activate_counter() {
            if(this.counter_id != null) {
                this.deactivate_counter()
            }
            this.counter_id = window.setInterval(function(self:any){
                self.seconds_passed += 1
                if(self.seconds_passed >= 60) {
                    self.seconds_passed = 0
                    self.minutes_passed += 1
                }
                if(self.minutes_passed >= 60) {
                    self.minutes_passed = 0
                    self.hours_passed += 1
                }
                document.title = `Timecheck - ${self.hours_passed}:${self.minutes_passed}:${self.seconds_passed}`
            }, 1000, this)
            this.counting = true

        },
        deactivate_counter() {
            window.clearInterval(this.counter_id)
            this.counter_id = 0
            this.counting = false
            this.seconds_passed = 0
            this.minutes_passed = 0
            this.hours_passed = 0
            document.title = 'Timecheck'
        },
        clear_timestamps: function(){
            this.history += `----- Cleared at ${this.get_current_time()} -----\r\n${this.timestamps}\r\n`
            this.timestamps = ''
            this.deactivate_counter()
            this.select_timestamps_field()
        },
        copy_timestamps: function(){
            let text_field = <HTMLTextAreaElement>document.getElementById('timestamps')
            if (text_field){
                text_field.select()
                text_field.focus()
            }

            navigator.clipboard.writeText(this.timestamps).then(() => {
                console.log('Timestamps copied to clipboard')
            })
        },
        get_current_time() {
            const now = new Date()
            
            this.last_hour = now.getHours()
            this.last_minute = now.getMinutes()
            this.last_second = now.getSeconds()

            const datetime_values: { [id:string]: number } = {
                month: now.getMonth() +1,
                date: now.getDate(),
                hours: now.getHours(),
                minutes: now.getMinutes(),
                seconds: now.getSeconds(),
            }

            const datetime_strings: {[id:string]: string} = {}
            for(const value in datetime_values){
                // Prepends a '0' to all single-digit values
                if(datetime_values[value] < 10){
                    datetime_strings[value] = `0${datetime_values[value]}`
                }
                else{
                    datetime_strings[value] = datetime_values[value].toString()
                }
            }
            
            return `${now.getFullYear()}-${datetime_strings.month}-${datetime_strings.date} ` +
                `${datetime_strings.hours}:${datetime_strings.minutes}:${datetime_strings.seconds}`
        },
        clear_history() {
            this.history = ''
        },
        select_timestamps_field() {
            const timestamps = document.getElementById('timestamps')

            if(timestamps){
                timestamps.focus()
            }
        },
        get_passed_time() {
            let res = ''

            if(this.hours_passed >= 1){
                res += `${this.hours_passed}h `
            }
            if(this.minutes_passed >= 1){
                res += `${this.minutes_passed}m `
            }
            res += `${this.seconds_passed}s`

            return res
        },
    }
})
</script>

<style scoped>
textarea{
    width: 95%;
    min-width: 95%;
    max-width: 95%;
    height: 360px;
    min-height: 360px;
    max-height: 480px;
    margin-left: 10px;
}
.clock{
    font-size: 4em;
    margin: 1px;
}
</style>
