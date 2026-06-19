<template>
    <el-dialog v-model="dialog.calendarForm" :title="dialog.title" center :before-close="handleClear" width="600">
        <el-form @submit.prevent="submitForm" label-position="top">
            <el-form-item label="Calendar Name">
                <el-input v-model="calendarForm.calendarName" type="text" placeholder="Calendar Name"/>
            </el-form-item>
            <div class="submit_btn">
                <el-button type="primary">Confirm</el-button>
            </div>
        </el-form>
    </el-dialog>
</template>

<script>
import { ElMessage } from 'element-plus'
export default {
    props: {
        handleDialog: Object
    },
    data() {
        return {
             calendarForm: {
                userId: '70f13842-28d8-4e45-9d18-a043059816a3',
                calendarName: ''
            },

            dialog: {
                calendarForm: false,
                title: ''
            },
        }
    },
    methods: {
        async submitForm(){
            const { data, error } = await supabase
            .from('Calendar')
            .insert(this.calendarForm)

            if(error) {
                ElMessage.error('Failed to add Calendar')
                return
            }

            ElMessage.success('Calendar added successfully')
            this.$emit('getCalendarEvents')
        },

        handleClear() {
            this.$emit('handleClear')
            this.dialog.calendarForm = false
            this.calendarForm.calendarName = ''
            console.log('clear')
        }
    },

    watch: {
        handleDialog(newVal) {
            console.log(newVal)
        }

    }
}
</script>