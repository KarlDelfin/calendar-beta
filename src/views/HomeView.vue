<template>
  <div class="home_con">
    <div class="sidebar">

      <div class="vcalendar">
        <VCalendar 
           :key="pickerKey"
            style="width: 100%"
            v-model="today"
            @dayclick="dayClick"
            @did-move="didMove"
            :attributes="attributes"
        />
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
        <el-checkbox label="Option 1" size="large" />
      </div>

    </div>

    <div class="calendar">
      <FullCalendar class="fullCalendar" ref="refCalendar" :options="calendarOptions" />
    </div>
 </div>

  <el-dialog v-model="dialog.calendarForm" :title="dialog.title" center :before-close="clear" width="600">
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
import FullCalendar from '@fullcalendar/vue3'
import rrulePlugin from '@fullcalendar/rrule'
import dayGridPlugin from '@fullcalendar/daygrid'
import interactionPlugin from '@fullcalendar/interaction'
import timeGridPlugin from '@fullcalendar/timegrid'
import list from '@fullcalendar/list'
import { supabase } from '../lib/supabaseClient'
import { ElMessage } from 'element-plus'
import moment from 'moment'

export default {
  components: { FullCalendar, },
  data(){
    return {
      firstDayOfMonth: new Date(),
      lastDayOfMonth: new Date(),
      currentYear: new Date(),
      currentMonth: new Date(),
      currentStartDateTime: new Date(),
      currentEndDateTime: new Date(),

      weekClicked: false,
      dayClicked: false,

      calendarApi: null,

      pickerKey: 0,

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
      today: new Date(),


      // START FULLCALENDAR
      calendarOptions: {
         datesSet: (info) => {
          this.datesSet(info)
        },
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
              this.formController('Create Event')
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
        // validRange: function(nowDate) {
        //   return {
        //     start: nowDate
        //   };
        // },
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
    computed: {
      attributes() {
        return [
          // {
          //   dot: 'bar',
          //   dates: this.calendarOptions.events,
          // },
          {
            highlight: {
              start: { fillMode: 'outline' },
              base: { fillMode: 'light' },
              end: { fillMode: 'outline' },
            },
            dates: { start: new Date(this.firstDayOfMonth), end: new Date(this.lastDayOfMonth) },
          },
        ]
      },
    },
  methods: {
      clear(){
        this.dialog.calendarForm = false
      },

      formController(title){
        this.dialog.title = title
        if(title === 'Create Calendar'){
          this.dialog.calendarForm = true
        }
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

      datesSet(info) {
        this.currentStartDateTime = moment(info.view.activeStart).format()
        this.currentEndDateTime = moment(info.view.activeEnd).format()

        let lastDay = moment(info.view.activeEnd).subtract(1, 'days')

        this.firstDayOfMonth = moment(info.view.activeStart).format()
        this.lastDayOfMonth = lastDay.format()

        this.currentMonth = moment(info.view.activeStart).format('MM')
        this.currentYear = moment(info.view.activeStart).format('YYYY')
      },

      todayCustom() {
        this.pickerKey++
        this.today = new Date()
        this.calendarApi.today()
        this.getCalendarEvents()
      },
      
      prevCustom() {
          this.pickerKey++
          this.calendarApi.prev()
          const currentDate = moment(this.calendarApi.getDate()).format();
          this.calendarApi.gotoDate(currentDate)
          this.getCalendarEvents()
      },

      nextCustom() {
          this.pickerKey++
          this.calendarApi.next()

          const currentDate = moment(this.calendarApi.getDate()).format();
          this.calendarApi.gotoDate(currentDate)
          this.getCalendarEvents()

          console.log(this.pickerKey)
      },

      monthCustom() {
        this.weekClicked = false
        this.dayClicked = false
        this.calendarApi.changeView('dayGridMonth')
        this.getCalendarEvents()
      },

      weekCustom() {
        this.weekClicked = true
        this.dayClicked = false
        this.calendarApi.changeView('timeGridWeek')
        this.getCalendarEvents()
      },

      dayCustom() {
        this.weekClicked = false
        this.dayClicked = true
        this.calendarApi.changeView('timeGridDay')
        this.getCalendarEvents()
      },

      listCustom() {
        this.calendarApi.changeView('listMonth')
        this.getCalendarEvents()
      },

      getCalendarEvents(){
        console.log('EVENTS HERE...')
      },

      didMove(info) {
        let formatDate = moment(info[0].id).add(1, 'days').format()
        this.calendarApi.gotoDate(formatDate)
        this.getCalendarEvents()
      },

      dayClick(info) {
        this.weekClicked = false
        this.dayClicked = true
        this.calendarApi.changeView('timeGridDay')
        this.calendarApi.gotoDate(info.endDate)
        this.getCalendarEvents()
      },
  },
  mounted(){
    this.calendarApi = this.$refs.refCalendar.getApi()
    this.getCalendars()
  }
}
</script>