<template>
    <div>
        <div>
            <ul class="instructions">
                <li><span class="code-like">F1</span>, <span class="code-like">F2</span> and <span class="code-like">ESC</span> only work if the textarea beneath them is selected.</li>
                <li><span class="code-like">F1</span> logs a new timestamp and triggers a counter, next to the clock, that shows how much time has passed <i>since the last timestamp was logged</i> (So it can appear out of sync with the clock).</li>
                <li><span class="code-like">Log Time Difference</span> enables logging the number of seconds, minutes and hours that have passed since the last timestamp was logged.</li>
                <li><span class="code-like">F2</span> copies logged timestamps to the system clipboard.</li>
                <li><span class="code-like">ESC</span> deletes logged timestamps but saves a copy under the history tab.</li>
                <li><span class="code-like">Clear History</span> deletes the timestamps permanently.</li>
            </ul>
        </div>
    </div>

</template>
<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({name: 'Instructions'})
</script>
<style scoped>
.code-like{
    color: green;
    background-color: rgb(230, 230, 230);
    padding: 3px;
    border: solid 1px;
    border-radius: 4px;
}
.instructions{
    text-align: left;
}
.instructions li{
    line-height: 2em;
}
</style>