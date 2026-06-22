<template>
    <div class="back_btn_con">
        <el-button @click="this.$router.push('/')">Back</el-button>
    </div>
    <div class="header_con">
        <h2>Calendar</h2>
        <el-button type="primary" @click="formController('Create Calendar')">Create Calendar</el-button>
    </div>

    <!-- CALENDAR SEARCH -->
    <el-collapse accordion>
        <el-collapse-item title="Filter Results" name="1">
        <el-form @submit.prevent="getCalendarsByUserId">
            <el-input placeholder="Search Calendar" v-model="search.calendarName" />
            <div class="search_con">
                <el-button @click="clear(), getCalendarsByUserId()" :loading="loading"> Reset </el-button>
                <el-button type="primary" :loading="loading" @click="getCalendarsByUserId"> Apply </el-button>
            </div>
        </el-form>
        </el-collapse-item>
    </el-collapse>

    <!-- CALENDAR TABLE -->
    <div v-if="calendars.length == 0">
        <el-empty description="No Data" />
    </div>
    <el-table v-else :data="calendars">
        <el-table-column prop="dateTimeCreated" label="Date/Time Created"/>
        <el-table-column prop="calendarName" label="Calendar Name"/>
        <el-table-column label="Share Calendar">
            <template #default="scope">
                <el-button @click="formController('Share Calendar', scope.row)">View Users</el-button>
            </template>
        </el-table-column>
            
        <el-table-column label="Operation" align="center">
            <template #default="scope">
                <el-button @click="formController('Edit Calendar', scope.row)">Edit</el-button>
                <el-button @click="deleteCalendar(scope.row.calendarId)" type="danger">Delete</el-button>
            </template>
        </el-table-column>
    </el-table>

    <!-- CALENDAR TABLE PAGINATION -->
    <el-pagination
        v-model:current-page="calendarPagination.currentPage"
        v-model:page-size="calendarPagination.elementsPerPage"
        :page-sizes="[5, 10, 25, 50]"
        :total="calendarPagination.totalElements"
        layout="total, sizes, prev, pager, next, jumper"
        @current-change="getCalendarsByUserId"
        @size-change="getCalendarsByUserId"
      />

    <!-- CALENDAR FORM -->
    <el-dialog v-model="dialog.calendarForm" :title="dialog.title" center :before-close="clear" width="600">
        <el-form label-position="top">
            <el-form-item label="Calendar Name">
            <el-input v-model="calendarForm.calendarName" type="text" placeholder="Calendar Name"/>
            </el-form-item>
            <div class="submit_btn">
            <el-button @click="submitForm" type="primary" :loading="loading">Confirm</el-button>
            </div>
        </el-form>
    </el-dialog>

    <!-- SHARE CALENDAR DIALOG -->
    <el-dialog v-model="dialog.sharedCalendarForm" :title="dialog.title" center :before-close="clear" width="600">
        <el-form label-position="top">
            <el-collapse accordion>
                <el-collapse-item title="Filter Results" name="1">
                <el-form>
                    <el-input placeholder="Search User" v-model="search.userEmail" />
                    <div class="search_con">
                        <el-button @click="clear"> Reset </el-button>
                        <el-button type="primary" @click="getUsersByEmail" :loading="loading"> Apply </el-button>
                    </div>
                </el-form>
                </el-collapse-item>
            </el-collapse>
            
            <!-- SHARED CALENDAR TABLE -->
            <div v-if="users.length == 0">
                <el-empty description="No Data" />
            </div>
            <el-table v-else :data="users" :loading="loading">
                <el-table-column label="User Name">
                    <template #default="scope">
                        <p>{{ scope.row.firstName }} {{ scope.row.lastName }}</p>
                    </template>
                </el-table-column>
                <el-table-column prop="email" label="Email"/>
                <el-table-column label="Operation">
                    <template #default="scope" align="end">
                        <el-button @click="assignUser(scope.row)" type="primary" :loading="loading">Assign</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-form>
    </el-dialog>
</template>

<script>
import { ElMessage, ElMessageBox, ElLoading } from 'element-plus';
import { supabase } from '@/lib/supabaseClient';
import moment from 'moment';
export default {
    data() {
        return {
            selectedCalendarId: '',
            users: [],
            calendars: [],
            calendarPagination: {
                currentPage: 1,
                elementsPerPage: 10,
                totalElements: 0
            },

            sharedUsersPagination: {
                currentPage: 1,
                elementsPerPage: 10,
                totalElements: 0
            },

            dialog: {
                sharedCalendarForm: false,
                calendarForm: false,
                title: '',
            },

            calendarForm: {
                userId: '70f13842-28d8-4e45-9d18-a043059816a3',
                calendarId: '',
                calendarName: ''
            },

            sharedCalendarForm: {
                calendarId: '',
                userId: ''
            },

            loading: false,

            search: {
                calendarName: '',
                userEmail: ''
            }
        }
    },
    methods: {
        /* FORM CONTROLLER */
        async formController(title, data){
            this.dialog.title = title
            if(title === 'Create Calendar'){
                this.dialog.calendarForm = true
            }

            if(title === 'Edit Calendar'){
                this.dialog.calendarForm = true
                this.calendarForm.calendarName = data.calendarName
                this.calendarForm.calendarId = data.calendarId
            }

            if(title === 'Share Calendar'){
                this.dialog.sharedCalendarForm = true
                this.selectedCalendarId = data.calendarId
                console.log(data)
            }
        },
        
        /* SUBMIT FORM */
        async submitForm() {
            /* CREATE CALENDAR */
            if (this.dialog.title === 'Create Calendar') {
                this.loading = true
                let payload = {
                    userId: this.calendarForm.userId,
                    calendarName: this.calendarForm.calendarName
                }

                try {
                    const { data, error } = await supabase
                        .from('Calendar')
                        .insert(payload)

                    if (error) {
                        ElMessage.error('Failed to add Calendar')
                        return
                    }

                    ElMessage.success('Calendar added successfully')
                    this.clear()
                    this.getCalendarsByUserId(this.calendarPagination.currentPage)
                } 
                catch (error) {
                    ElMessage.error('An unexpected error occurred')
                    console.error(error)
                } 
                finally {
                    this.loading = false
                }
            }

            if(this.dialog.title === 'Edit Calendar'){
                this.loading = true
                let payload = {
                    userId: this.calendarForm.userId,
                    calendarName: this.calendarForm.calendarName
                }
                try{
                    const { data, error} = await supabase
                        .from('Calendar')
                        .update(payload)
                        .eq('calendarId', this.calendarForm.calendarId)
                    if(error) throw error
                    ElMessage.success('Calendar updated successfully')
                    this.clear()
                    this.getCalendarsByUserId()
                }
                catch (error) {
                    ElMessage.error('An unexpected error occurred')
                    console.error(error)
                } 
                finally {
                    this.loading = false
                }
            }
        },

        /* GET CALENDARS BY USER ID */
        async getCalendarsByUserId() {
            this.loading = true
            const loading = ElLoading.service({
                lock: true,
                text: 'Loading',
                background: 'rgba(0, 0, 0, 0.7)',
            })

            try {
                const limit = this.calendarPagination.elementsPerPage;
                const from = (this.calendarPagination.currentPage - 1) * limit;
                const to = from + limit - 1;

                let query = supabase
                    .from('Calendar')
                    .select('*', { count: 'exact' })
                    .eq('userId', this.calendarForm.userId);

                if (this.search.calendarName && this.search.calendarName.trim() !== '') {
                    query = query.ilike('calendarName', `%${this.search.calendarName}%`);
                }

                query = query.order('dateTimeCreated', { ascending: false }).range(from, to);

                const { data, error, count } = await query;
                if (error) throw error;

                this.calendars = data;

                this.calendars = this.calendars.map((data)=> ({
                    calendarId: data.calendarId,
                    calendarName: data.calendarName,
                    dateTimeCreated: moment(data.dateTimeCreated).format('MMMM DD, YYYY HH:mm:ss')
                }))

                this.calendarPagination.currentPage = this.calendarPagination.currentPage;
                this.calendarPagination.totalElements = count || 0;

            } catch (error) {
                console.error(error);
            } finally {
                loading.close()
                this.loading = false
            }
        },

        /* DELETE CALENDAR */
        async deleteCalendar(calendarId){
            await ElMessageBox.confirm('Do you want to delete this Calendar?', 'Warning', { confirmButtonText: 'OK', cancelButtonText: 'Cancel', type: 'warning', } )

            const loading = ElLoading.service({
                lock: true,
                text: 'Loading',
                background: 'rgba(0, 0, 0, 0.7)',
            })

            try{
                const {data, error} = await supabase
                .from('Calendar')
                .delete()
                .eq('calendarId', calendarId)

                if(error){
                    ElMessage.error("Failed to assign User")
                }
                ElMessage.success("Calendar deleted successfully")
                this.clear()
                await this.getCalendarEventsByCalendarId(this.selectedCalendarId)
            }
            catch(error){
                ElMessage.error('An unexpected error occurred')
                console.error(error)
            }
            finally{
                loading.close()
            }

        },

        /* GET USER BY EMAIL */
        async getUsersByEmail() {
            this.loading = true;
            
            try {
                const { data, error } = await supabase
                    .from('User')
                    .select('*')
                    .eq('email', this.search.userEmail)
                    .filter('userId', 'not.in', 
                        `(${await supabase.from('SharedCalendar').select('userId').then(res => res.data?.map(r => r.userId).join(','))})`
                    );
                
                if (error) throw error;
                this.users = data;
            } catch (error) {
                ElMessage.error('An unexpected error occurred');
                console.error(error);
            } finally {
                this.loading = false;
            }
        },

        /* ASSIGN CALENDAR TO A USER */
        async assignUser(data) {
            this.loading = true;
            let payload = {
                calendarId: this.selectedCalendarId,
                userId: data.userId
            }

            try{
                const { data, error } = await supabase
                    .from('SharedCalendar')
                    .insert(payload)
                if(error) throw error
                ElMessage.success("User assigned successfully")
                await this.getUsersByEmail()
            }
            catch(error){
                ElMessage.error('An unexpected error occurred')
                console.error(error)
            }
            finally{
                this.loading = false;
            }
        },

        clear(){
            this.dialog.calendarForm = false
            this.dialog.sharedCalendarForm = false
            this.dialog.title = ''

            this.calendarForm.calendarName = ''

            this.search.calendarName = ''
            this.search.userEmail = ''

            this.loading = false

            this.selectedCalendarId = ''
        },

        clearSearch(){
            this.search.calendarName = ''
            this.search.userEmail = ''
        }
    },
    mounted(){
        this.getCalendarsByUserId()
    }
}
</script>