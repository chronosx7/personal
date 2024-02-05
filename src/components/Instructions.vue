<template>
    <div style="padding: 20px;">
        <div>
            <ul class="instructions">
                <li><span class="code-like">F1</span> logs a new timestamp and triggers a counter, next to the clock, that shows how much time has passed <i>since the last timestamp was logged</i> (So it can seem out of sync with the clock).</li>
                <li><span class="code-like">Copy</span> copies logged timestamps to the system clipboard.</li>
                <li><span class="code-like">ESC</span> deletes logged timestamps.</li>
                <!-- <li><span class="code-like">ESC</span> deletes logged timestamps but saves a copy under the history tab.</li> -->
                <!-- <li><span class="code-like">Clear History</span> deletes the timestamps permanently.</li> -->
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