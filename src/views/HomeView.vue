<template>
  <div class="home_con">
    <div class="sidebar">

      <div class="vcalendar">
        <VCalendar/>
      </div>

      <div class="create_calendar">
        <el-button @click="formController('Create Calendar')" type="primary">Create Calendar</el-button>
      </div>

      <div class="calendar_select">
        <el-select v-model="sidebar.calendarId">
          <el-option v-for="calendar in calendars" :label="calendar.calendarName" :value="calendar.calendarId"/>
        </el-select>
      </div>

      <div class="calendar_events_checkbox">
        <el-checkbox v-model="checked1" label="Option 1" size="large" />
      </div>

    </div>

    <div class="calendar">
      <FullCalendar class="fullCalendar" ref="refCalendar" :options="calendarOptions" />
    </div>
 </div>

  <el-dialog>
    <el-form @submit.prevent="submitForm">
      <el-form-item>
        <el-input v-model="calendarForm.calendarName" type="text" />
      </el-form-item>
      <el-button type="primary">Confirm</el-button>
    </el-form>
  </el-dialog>
</template>

<script>
import FullCalendar from '@fullcalendar/vue3'
import rrulePlugin from '@fullcalendar/rrule'
import dayGridPlugin from '@fullcalendar/daygrid'
import interactionPlugin from '@fullcalendar/interaction'
import timeGridPlugin from '@fullcalendar/timegrid'
import list from '@fullcalendar/list'
import { supabase } from '../lib/supabaseClient'
import { ElMessage } from 'element-plus'

export default {
  components: { FullCalendar, },
  data(){
    return {
      sidebar: {
        calendarId: ''
      },
      dialog: {
        calendarForm: false,
        title: ''
      },

      calendars: [],
      calendarForm: {
        userId: '70f13842-28d8-4e45-9d18-a043059816a3',
        calendarName: ''
      },
      now: new Date(),


      // START FULLCALENDAR
      calendarOptions: {
        eventTimeFormat: {
          hour: '2-digit',
          minute: '2-digit',
          meridiem: false,
          hour12: false,
        },
        views: {
          dayGridMonth: {
            dayMaxEventRows: 2,
            titleFormat: { year: 'numeric', month: 'long', day: 'numeric' },
          },

          timeGridWeek: {
            slotLabelFormat: {
              hour: '2-digit',
              minute: '2-digit',
              hour12: false, // 24-hour format
            },
            eventTimeFormat: {
              hour: '2-digit',
              minute: '2-digit',
              hour12: false, // 24-hour format
            },
          },

          timeGridDay: {
            slotLabelFormat: {
              hour: '2-digit',
              minute: '2-digit',
              hour12: false, // 24-hour format
            },
          },

          listMonth: {
            eventTimeFormat: {
              hour: '2-digit',
              minute: '2-digit',
              hour12: false, // 24-hour format
            },
          },
        },
        headerToolbar: {
          start: 'todayCustom,prevCustom,nextCustom',
          center: 'title',
          end: 'createEvent monthCustom,weekCustom,dayCustom,listCustom',
        },
        // Custom Button
        customButtons: {
          createEvent: {
            text: 'Create Event',
            click: () => {
              this.openForm('Create Event')
            },
          },
          todayCustom: {
            text: 'today',
            click: () => {
              this.todayCustom()
            },
          },
          prevCustom: {
            text: 'prev',
            click: () => {
              this.prevCustom()
            },
          },
          nextCustom: {
            text: 'next',
            click: () => {
              this.nextCustom()
            },
          },
          monthCustom: {
            text: 'month',
            click: () => {
              this.monthCustom()
            },
          },
          weekCustom: {
            text: 'week',
            click: () => {
              this.weekCustom()
            },
          },
          dayCustom: {
            text: 'day',
            click: () => {
              this.dayCustom()
            },
          },
          listCustom: {
            text: 'list',
            click: () => {
              this.listCustom()
            },
          },
        },
        height: 600,
        plugins: [dayGridPlugin, interactionPlugin, timeGridPlugin, list, rrulePlugin],
        timeZone: 'UTC',
        events: [],
        firstDay: 0, // 0 - [Mon, Tue, Wed, Thu, Fri, Sat, Sun] | 1 - [Sun, Mon, Tue, Wed, Thu, Fri, Sat]
        initialView: 'dayGridMonth', // Default fullcalendar view
        eventDrop: this.moveResizeEvent, // Drag and drop
        eventResize: this.moveResizeEvent, // Resize event
        dateClick: this.handleClick, // click single date
        selectable: true, // Set selectable to true
        editable: true, // Set editable to true
        eventResizableFromStart: true, // Set resizable to true
        droppable: true, // Set drag and drop to true

        select: this.handleDateRange, // Select multiple event
        eventClick: this.eventClick, // Click event
        allDaySlot: false, // Show/hide non-recurring events false
        eventLongPressDelay: 200, // Mobile drog n drop events
        eventOverlap: true,
        forceEventDuration: true,
        displayEventTime: true, // Display time for recurring event
        showNonCurrentDates: false, // Disable last week of previous month and first week of next month
      },
    }
  },
  methods: {
      formController(title){

      },
      async submitForm(){
        const { data, error } = await supabase
        .from('Calendar')
        .insert(this.calendarForm)

        if(error) {
          ElMessage.error('Failed to add Calendar')
          return
        }

        ElMessage.success('Calendar added successfully')
        this.getCalendars()
      },

      async getCalendars() {
        const { data, error } = await supabase
          .from('Calendar')
          .select('*')

        this.calendars = data
      },

      async moveResizeEvent(info){console.log(info)},
      async handleClick(info){console.log(info)},
      async handleDateRange(info){console.log(info)},
      async eventClick(info){console.log(info)},

      async todayCustom(info) {console.log(info)},
      async prevCustom(info) {console.log(info)},
      async nextCustom(info) {console.log(info)},
      async nextCustom(info) {console.log(info)},
      async monthCustom(info) {console.log(info)},
      async dayCustom(info) {console.log(info)},
      async listCustom(info) {console.log(info)},
  },
  mounted(){
    this.getCalendars()
  }
}
</script>