<template>
    <v-table :height="height" fixed-header>
        <thead>
            <tr>
                <th></th>
                <th v-for="attr in headers">
                    {{ attr }}
                </th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="(entry, row) in sourceList"
                @keyup.down="up_down($event, Dir.DOWN)"
                @keyup.up="up_down($event, Dir.UP)"
            >
                <td>{{ row + 1 }}</td>
                <td v-for="(val, attr, col) in entry">
                    <input :value="val" class="filling-textbox"
                        :data-col="col+1"
                        @keyup.right="left_right($event, Dir.RIGHT)"
                        @keyup.left="left_right($event, Dir.LEFT)"
                        @input="update_row(($event.target as HTMLInputElement).value, attr, row)"
                    >
                </td>
                <td v-show="show_actions" style="max-width: 25px;">
                    <v-btn variant="text" block icon="mdi-minus-circle-outline" @click="remove_row(row)"/>
                </td>
            </tr>
        </tbody>
    </v-table>
</template>
<script setup lang="ts">
    import { ref } from 'vue';

    const props = defineProps(['sourceList', 'headers', 'height'])
    const emit = defineEmits(['rowRemoved'])
    const show_actions = ref(true)

    enum Dir{UP, DOWN, RIGHT, LEFT}

    let caret_pos = -1

    function up_down(e:KeyboardEvent, dir:Dir){
        const elem = e.target as HTMLElement
        const target_col = Number(elem.attributes.getNamedItem('data-col')?.value) + 1
        let new_row

        switch(dir){
            case Dir.DOWN:{
                new_row = elem.parentElement?.parentElement?.nextElementSibling
                break
            }
            case Dir.UP:{
                new_row = elem.parentElement?.parentElement?.previousElementSibling
                break
            }
        }
        if(new_row){
            const new_cell = new_row.childNodes[target_col].childNodes[0] as HTMLElement
            new_cell.focus()
        }
    }
    
    function left_right(e:KeyboardEvent, dir:Dir){
        const elem = e.target as HTMLFormElement
        const new_caret_pos = elem.selectionEnd
        const source_col = Number(elem.attributes.getNamedItem('data-col')?.value)
        
        if(caret_pos === new_caret_pos){
            let new_cell = undefined
            switch(dir){
                case Dir.RIGHT:{
                    new_cell = source_col < props.headers.length? elem.parentElement?.nextElementSibling: undefined
                    break
                }
                case Dir.LEFT:{
                    new_cell = source_col > 1? elem.parentElement?.previousElementSibling: undefined
                    break
                }
            }
            if(new_cell != undefined){
                const form_elem = new_cell.childNodes[0] as HTMLFormElement
                form_elem.focus()
                caret_pos = -1
                form_elem.selectionEnd = form_elem.selectionStart = dir == Dir.RIGHT? 0: form_elem.textLength
            }
        }
        caret_pos = new_caret_pos
    }

    function remove_row(row:number){
        props.sourceList.splice(row, 1)
        emit('rowRemoved')
    }

    function update_row(value:string, attr:number, row:number){
        props.sourceList[row][attr as keyof typeof props.sourceList.value] = value
    }

</script>
<style>
    .filling-textbox{
        width: 100%;
        height: 90%;
        margin: 0;
        padding-left: 5px;
    }
    .action-button{
        border-radius: 5px;
        border: solid 1px;
        padding: 5px;
        margin: 0 2px;
        min-width: 25px;
    }
    .bordered{
        border: solid 1px;
    }
    th, td{
        border-right: solid 1px rgb(220, 220, 220);
    }
    table{
        border: 1px solid rgb(200, 200, 200);
    }
    th{
        text-align: center !important;
        background-color: rgb(245, 248, 250) !important;
        font-size: 1.3em;
        font-weight: 600;
    }
    td{
        background-color: white;
    }
</style>