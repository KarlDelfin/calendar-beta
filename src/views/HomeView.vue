<template>
  <div class="home_con">
    <div class="sidebar">
      <div class="cta">
        <el-button @click="this.$router.push('/calendar')">Calendars</el-button>
      </div>

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
        <el-select v-model="sidebar.calendarId" :loading="loading.selectCalendarInput" @click="getCalendars" @change="getCalendarEventsByCalendarId">
          <el-option v-for="calendar in calendars" :label="calendar.calendarName" :value="calendar.calendarId"/>
        </el-select>
      </div>

      <div class="calendar_events_checkbox">
        <el-checkbox 
          v-model="selectedCalendarEvent[calendarEvent.calendarEventId]"
          v-for="calendarEvent in calendarEvents" 
          :key="calendarEvent.calendarEventId"
          :label="calendarEvent.calendarEventName" 
          :value="calendarEvent.calendarEventId" 
          @change="handleCheckboxChange(calendarEvent.calendarEventId, calendarEvent.calendarEventGroupId)"
          size="large" 
          :checked="true"
          />
      </div>

      <div class="show_hide_events">
          <el-button @click="checkAllEvents" >Select All</el-button >
          <el-button @click="uncheckAllEvents" >Select None</el-button >
      </div>

    </div>

    <div class="calendar">
      <FullCalendar class="fullCalendar" ref="refCalendar" :options="calendarOptions" />
    </div>
 </div>

  <!-- CALENDAR FORM -->
  <el-dialog v-model="dialog.calendarForm" :title="dialog.title" center :before-close="clear" width="600">
      <el-form label-position="top">
        <el-form-item label="Calendar Name">
          <el-input v-model="calendarForm.calendarName" type="text" placeholder="Calendar Name"/>
        </el-form-item>
        <div class="submit_btn">
          <el-button @click="submitForm" type="primary" :loading="loading.calendarSubmitBtn">Confirm</el-button>
        </div>
      </el-form>
  </el-dialog>
  
  <!-- CALENDAR EVENT FORM -->
  <el-dialog v-model="dialog.calendarEventForm" :title="dialog.title" center :before-close="clear" width="600">
    <el-form label-position="top">
      <el-form-item label="Calendar Name">
        <el-select v-model="sidebar.calendarId" disabled>
            <el-option v-for="calendar in calendars" :label="calendar.calendarName" :value="calendar.calendarId"/>
        </el-select>
      </el-form-item>
      <el-form-item label="Event Name">
        <el-input v-model="calendarEventForm.calendarEventName" type="text" placeholder="Event Name"/>
      </el-form-item>
      <el-form-item label="Event Description">
        <el-input v-model="calendarEventForm.calendarEventDescription" type="textarea" placeholder="Event Description"/>
      </el-form-item>
      
      <div class="flex_con">
        <el-form-item label="Event Color">
          <el-color-picker v-model="calendarEventForm.calendarEventColor" :predefine="predefineColors"/>
        </el-form-item>
        <el-form-item label="Is Recurring?">
          <el-radio-group v-model="calendarEventForm.isRecurring" @change="calendarEventForm.selectedRecurringDays = []">
            <el-radio label="Yes" :value="true" />
            <el-radio label="No" :value="false" />
          </el-radio-group>
        </el-form-item>
      </div>
      <el-divider content-position="left">Date Range</el-divider>
      <el-form-item label="Date Range">
        <VDatePicker
          :popover="popover"
          v-model.range="calendarEventForm.dateRange"
          :columns="2"
          :first-day-of-week="1"
          mode="date"
          :min-date="new Date()"
        >
          <template #default="{ inputValue, inputEvents }">
            <div class="flex_con">
                <el-input
                  :value="inputValue.start"
                  v-on="inputEvents.start"
                  placeholder="Start Date"/>

                <el-input 
                  :value="inputValue.end" 
                  v-on="inputEvents.end" 
                  placeholder="End Date" />
              </div>
          </template>
        </VDatePicker>
      </el-form-item>
      <div class="flex_con">
        <el-form-item label="Start Time">
          <el-time-picker
            :disabled="dayClicked || weekClicked"
            format="HH:mm"
            v-model="calendarEventForm.startTime"
            placeholder="Start Time"
          />
        </el-form-item>
        <el-form-item label="End Time">
          <el-time-picker
            :disabled="dayClicked || weekClicked"
            format="HH:mm"
            v-model="calendarEventForm.endTime"
            placeholder="End Time"
          />
        </el-form-item>
      </div>

      <div v-if="calendarEventForm.isRecurring">
        <el-divider content-position="left">Recurring Every</el-divider>
        <el-checkbox
          v-for="(day, index) in recurringDays"
          :key="index"
          @change="selectedRecurringDay(day.value)"
          :label="day.label"
          :value="day.value"
          size="large"
          :checked="day.checked"
          :disabled="day.disabled"
        >
          {{ day.label }}
        </el-checkbox>
      </div>
      <div class="submit_btn">
        <el-button @click="submitForm" type="primary" :loading="loading.calendarEventSubmitBtn">Confirm</el-button>
      </div>
    </el-form>
  </el-dialog>

  <!-- RECURRING EVENT CONFIMATION DIALOG -->
  <el-dialog
    v-model="dialog.calendarEventConfirmation"
    :title="dialog.title"
    width="500"
    center
    :before-close="clear"
  >
    <div class="d-flex justify-content-center mt-3">
      <span>This is one activity in a series. What do you want to open?</span>
    </div>
    <!-- RADIO BUTTON -->
    <div class="pt-3 d-flex justify-content-center text-center text-sm">
      <el-radio-group v-model="operationForm.isSeries" class="ml-4">
        <el-radio label="Just this one" :value="false" size="large" />
        <el-radio label="The entire series" :value="true" size="large" />
      </el-radio-group>
    </div>

    <template #footer>
      <div class="dialog-footer d-flex justify-content-end">
        <el-button type="primary" @click="formController('Event Details')">
          Confirm
        </el-button>
      </div>
    </template>
  </el-dialog>

   <!-- START OPERATION -->
    <el-dialog
      :before-close="clear"
      v-model="dialog.recurringEventOperation"
      :title="dialog.title"
      width="500"
      center
    >
      <div class="border p-3" style="border-radius: 20px">
        <label class="fw-bold">Event Name: </label>
        {{ calendarEventForm.calendarEventName }}
        <br />
        <label class="fw-bold mt-3">Description: </label>
        {{ calendarEventForm.calendarEventDescription }}
        <br />
        <label class="fw-bold mt-3">Date/Time Start: </label>
        {{ calendarEventForm.dateTimeStartedDetails }}
        <br />
        <label class="fw-bold mt-3">Date/Time End: </label>
        {{ calendarEventForm.dateTimeEndedDetails }}
      </div>

      <template #footer>
        <div class="dialog-footer d-flex justify-content-end">
          <el-button @click="formController('Edit Non-Recurring Event')"> Edit </el-button>
          <el-button type="danger" @click="deleteCalendarEvent()"> Delete </el-button>
        </div>
      </template>
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
import { ElMessage, ElLoading, ElMessageBox  } from 'element-plus'
import moment from 'moment'
import { v4 as uuidv4 } from 'uuid'
import * as bootstrap from 'bootstrap'

export default {
  components: { FullCalendar, },
  data(){
    return {
      pickerKey: 0,
      popover: { visibility: 'click', },

      selectedCalendarId: '',
      selectedCalendarName: '',

      firstDayOfMonth: new Date(),
      lastDayOfMonth: new Date(),
      currentYear: new Date(),
      currentMonth: new Date(),
      currentStartDateTime: new Date(),
      currentEndDateTime: new Date(),

      weekClicked: false,
      dayClicked: false,

      

      sidebar: {
        calendarId: ''
      },

      dialog: {
        recurringEventOperation: false,
        calendarForm: false,
        calendarEventForm: false,
        calendarEventConfirmation: false,
        title: '',
      },

      loading: {
        calendarSubmitBtn: false,
        selectCalendarInput: false,
        calendarEventSubmitBtn: false
      },

      calendars: [],
      selectedCalendarEvent: [],
      calendarEvents: [],

      operationForm: {
        isSeries: false,
      },

      calendarForm: {
        userId: '70f13842-28d8-4e45-9d18-a043059816a3',
        calendarName: ''
      },
      calendarEventForm: {
        dateTimeStartedDetails: new Date(),
        dateTimeEndedDetails: new Date(),

        calendarEventId: '',
        calendarId: '',
        createdByUserId: '70f13842-28d8-4e45-9d18-a043059816a3',
        calendarEventName: '',
        calendarEventDescription: '',
        calendarEventColor: '#409EFF',
        startTime: new Date(2026, 6, 24, 9, 0),
        endTime: new Date(2026, 6, 24, 18, 0),
        isRecurring: false,
        calendarEventGroupId: '',
        dateRange: {
          start: null,
          end: null
        },

        selectedRecurringDays: []
      },
    
      recurringDays: [
        { label: 'Sun', value: 0, checked: false, disabled: false },
        { label: 'Mon', value: 1, checked: false, disabled: false },
        { label: 'Tue', value: 2, checked: false, disabled: false },
        { label: 'Wed', value: 3, checked: false, disabled: false },
        { label: 'Thu', value: 4, checked: false, disabled: false },
        { label: 'Fri', value: 5, checked: false, disabled: false },
        { label: 'Sat', value: 6, checked: false, disabled: false },
      ],

      predefineColors: [ '#ff4500', '#ff8c00', '#ffd700', '#90ee90', '#00ced1', '#1e90ff', '#c71585', ],



      
      today: new Date(),

      calendarApi: null,

      // START FULLCALENDAR
      calendarOptions: {
         datesSet: (info) => {
          var currentDate = info.view.activeStart; 

          this.currentStartDateTime = moment(info.view.activeStart).format()
          this.currentEndDateTime = moment(info.view.activeEnd).format()

          this.firstDayOfMonth = moment(currentDate).startOf('month').format();
          this.lastDayOfMonth = moment(currentDate).endOf('month').format();

          this.currentMonth = moment(info.view.activeStart).format('MM')
          this.currentYear = moment(info.view.activeStart).format('YYYY')
        },

        eventTimeFormat: {
          hour: '2-digit',
          minute: '2-digit',
          meridiem: false,
          hour12: false,
        },

        eventDidMount: (info) => {
          let description = info.event.extendedProps.calendarEventDescription
          let startTime = info.event.extendedProps.startTime
          let endTime = info.event.extendedProps.endTime
          return new bootstrap.Popover(info.el, {
            title: info.event.title,
            placement: 'auto',
            trigger: 'hover',
            customClass: 'popoverStyle',
            content: `
                <span class="fst-italic">${description}</span>
                <hr>
                <div class="d-flex justify-content-center">
                  <small class='fw-bold'>${startTime} - ${endTime}</small>
                </div>
                     `,
            html: true,
          })
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

        // validRange: function(nowDate) {
        //   return {
        //     start: nowDate
        //   };
        // },

        // CUSTOM BUTTONS
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
              this.pickerKey++
              this.today = new Date()
              this.calendarApi.today()
              this.getCalendarEventsByCalendarId(this.selectedCalendarId)
            },
          },

          prevCustom: {
            text: 'prev',
            click: () => {
              this.pickerKey++
              this.calendarApi.prev()
              const currentDate = moment(this.calendarApi.getDate()).format();
              this.calendarApi.gotoDate(currentDate)
              this.getCalendarEventsByCalendarId(this.selectedCalendarId)
            },
          },

          nextCustom: {
            text: 'next',
            click: () => {
              this.pickerKey++
              this.calendarApi.next()

              const currentDate = moment(this.calendarApi.getDate()).format();
              this.calendarApi.gotoDate(currentDate)
              this.getCalendarEventsByCalendarId(this.selectedCalendarId)
            },
          },

          monthCustom: {
            text: 'month',
            click: () => {
              this.weekClicked = false
              this.dayClicked = false
              this.calendarApi.changeView('dayGridMonth')
              this.getCalendarEventsByCalendarId(this.selectedCalendarId)
            },
          },

          weekCustom: {
            text: 'week',
            click: () => {
              this.weekClicked = true
              this.dayClicked = false
              this.calendarApi.changeView('timeGridWeek')
              this.getCalendarEventsByCalendarId(this.selectedCalendarId)
            },
          },

          dayCustom: {
            text: 'day',
            click: () => {
              this.weekClicked = false
              this.dayClicked = true
              this.calendarApi.changeView('timeGridDay')
              this.getCalendarEventsByCalendarId(this.selectedCalendarId)
            },
          },

          listCustom: {
            text: 'list',
            click: () => {
              this.calendarApi.changeView('listMonth')
              this.getCalendarEventsByCalendarId(this.selectedCalendarId)
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
        selectable: true, // Set selectable to true
        editable: true, // Set editable to true
        eventResizableFromStart: true, // Set resizable to true
        droppable: true, // Set drag and drop to true

        select: this.handleDateRange, // Select dates

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
    /* FORM CONTROLLER */
    async formController(title){
      this.dialog.title = title
      if(title === 'Create Calendar'){
        this.dialog.calendarForm = true
      }

      if(title === 'Create Event'){
        if(this.selectedCalendarId === '') {
          ElMessage.warning('Please select a calendar first')
          return
        }
        this.dialog.calendarEventForm = true
      }

      if(title === 'Edit Non-Recurring Event') {
        this.dialog.calendarEventForm = true
        this.dialog.recurringEventOperation = false

      }

      if(title === 'Edit Recurring Event') {
        this.dialog.calendarEventForm = true
        this.dialog.recurringEventOperation = false
      }

      if(title === 'Open Recurring Event') {
        this.dialog.recurringEventOperation = false
        this.dialog.calendarEventConfirmation = true
      }

      if(title === 'Event Details') {
        /* NON-RECURRING */
        this.dialog.recurringEventOperation = true

                
        /* RECURRING (ON SERIES) */
        if(this.operationForm.isSeries) {
          console.log("SERIESSSSSSSS!!!!!!!!!!!!!!!!!!!!")
          const { data, error } = await supabase
            .from('CalendarEvent')
            .select('*')
            .eq('calendarEventGroupId', this.calendarEventForm.calendarEventGroupId)   
            .order('dateTimeStarted', { ascending: true })

            const startDate = moment(data[0].dateTimeStarted)
            const endDate = moment(data[0].dateTimeEnded)

            this.calendarEventForm.dateTimeStartedDetails = startDate.format('MM/DD/YYYY') + ' ' + startDate.format('HH:mm')
            this.calendarEventForm.dateTimeEndedDetails = startDate.format('MM/DD/YYYY') + ' ' + endDate.format('HH:mm') 

            this.calendarEventForm.dateTimeStartedDetails = moment(data[0].dateTimeStarted).format('MM/DD/YYYY HH:mm')
            this.calendarEventForm.dateTimeEndedDetails = moment(data[0].dateTimeEnded).format('MM/DD/YYYY HH:mm')
            
            this.calendarEventForm.calendarEventId = data[0].calendarEventId
            this.calendarEventForm.calendarEventName = data[0].calendarEventName
            this.calendarEventForm.calendarEventDescription = data[0].calendarEventDescription
            this.calendarEventForm.calendarEventColor = data[0].calendarEventColor
            this.calendarEventForm.startTime = data[0].dateTimeStarted
            this.calendarEventForm.endTime = data[0].dateTimeEnded
            this.calendarEventForm.dateRange = { 
              start: moment(data[0].dateTimeStarted).format(),
              end:  moment(data[0].dateTimeEnded) .format()
              }
            this.calendarEventForm.isRecurring = data[0].isRecurring
            this.calendarEventForm.calendarId = data[0].calendarId
        }

        /* RECURRING (JUST ONE) */
        else {
          const { data, error } = await supabase
            .from('CalendarEvent')
            .select('*')
            .eq('calendarEventId', this.calendarEventForm.calendarEventId)  
            .order('dateTimeStarted', { ascending: false }) 

            const startDate = moment(data[0].dateTimeStarted)
            const endDate = moment(data[0].dateTimeEnded)

            this.calendarEventForm.dateTimeStartedDetails = startDate.format('MM/DD/YYYY') + ' ' + startDate.format('HH:mm')
            this.calendarEventForm.dateTimeEndedDetails = startDate.format('MM/DD/YYYY') + ' ' + endDate.format('HH:mm') 

            this.calendarEventForm.calendarEventId = data[0].calendarEventId
            this.calendarEventForm.calendarEventName = data[0].calendarEventName
            this.calendarEventForm.calendarEventDescription = data[0].calendarEventDescription
            this.calendarEventForm.calendarEventColor = data[0].calendarEventColor
            this.calendarEventForm.startTime = data[0].dateTimeStarted
            this.calendarEventForm.endTime = data[0].dateTimeEnded
            this.calendarEventForm.dateRange = { 
              start: startDate.format('YYYY-MM-DD') + 'T' + startDate.format('HH:mm:ss'),
              end:  startDate.format('YYYY-MM-DD') + 'T' + endDate.format('HH:mm:ss')
            }
            this.calendarEventForm.isRecurring = data[0].isRecurring
            this.calendarEventForm.calendarId = data[0].calendarId
        }
      }
    },

    /* CREATE / UPDATE */
    async submitForm() {
      /* CREATE CALENDAR */
      if (this.dialog.title === 'Create Calendar') {
        this.loading.calendarSubmitBtn = true
        
        try {
          const { data, error } = await supabase
            .from('Calendar')
            .insert(this.calendarForm)

          if (error) {
            ElMessage.error('Failed to add Calendar')
            return
          }

          ElMessage.success('Calendar added successfully')
          this.clear()
          this.getCalendars()
        } 
        catch (error) {
          ElMessage.error('An unexpected error occurred')
          console.error(error)
        } 
        finally {
          this.loading.calendarSubmitBtn = false
        }
      }

      /* CREATE CALENDAR EVENT */
      if(this.dialog.title === 'Create Event' || this.dialog.title === 'Edit Non-Recurring Event') {
        let calendarEvents = []
        let calendarEventGroupId = uuidv4()
        
        let startDate = new Date(moment(this.calendarEventForm.dateRange.start).format())
        let endDate = new Date(moment(this.calendarEventForm.dateRange.end).format())

        /* DELETE DATA FROM NON-RECURRING EVENT */
        if(this.dialog.title === 'Edit Non-Recurring Event') {
          const { data, error } = await supabase
            .from('CalendarEvent')
            .delete()
            .eq('calendarEventId', this.calendarEventForm.calendarEventId)
        }

        /* RECURRING */
        if(this.calendarEventForm.isRecurring) {
          while(startDate <= endDate) {
            const startDay = startDate.getDay()
            
            if(this.calendarEventForm.selectedRecurringDays.length > 0) {
              if(this.calendarEventForm.selectedRecurringDays.includes(startDay)) {
                calendarEvents.push({
                  calendarId: this.selectedCalendarId,
                  calendarEventName: this.calendarEventForm.calendarEventName,
                  createdByUserId: this.calendarEventForm.createdByUserId,
                  calendarEventDescription: this.calendarEventForm.calendarEventDescription,
                  calendarEventColor: this.calendarEventForm.calendarEventColor,
                  isRecurring: this.calendarEventForm.isRecurring,

                  dateTimeStarted: moment(startDate).format(`YYYY-MM-DDT${moment(this.calendarEventForm.startTime).format('HH:mm:ss')}`),
                  dateTimeEnded: moment(endDate).format(`YYYY-MM-DDT${moment(this.calendarEventForm.endTime).format('HH:mm:ss')}`),

                  calendarEventGroupId: calendarEventGroupId,
                })
              }
            }

            if(this.calendarEventForm.selectedRecurringDays.length === 0){
                  calendarEvents.push({
                  calendarId: this.selectedCalendarId,
                  calendarEventName: this.calendarEventForm.calendarEventName,
                  createdByUserId: this.calendarEventForm.createdByUserId,
                  calendarEventDescription: this.calendarEventForm.calendarEventDescription,
                  calendarEventColor: this.calendarEventForm.calendarEventColor,
                  isRecurring: this.calendarEventForm.isRecurring,

                  dateTimeStarted: moment(startDate).format(`YYYY-MM-DDT${moment(this.calendarEventForm.startTime).format('HH:mm:ss')}`),
                  dateTimeEnded: moment(endDate).format(`YYYY-MM-DDT${moment(this.calendarEventForm.endTime).format('HH:mm:ss')}`),

                  calendarEventGroupId: calendarEventGroupId,
                })
              }
            startDate.setDate(startDate.getDate() + 1)
          }
        }

        /* NON-RECURRING */
        else{
          calendarEvents.push({
            calendarId: this.selectedCalendarId,
            calendarEventName: this.calendarEventForm.calendarEventName,
            createdByUserId: this.calendarEventForm.createdByUserId,
            calendarEventDescription: this.calendarEventForm.calendarEventDescription,
            calendarEventColor: this.calendarEventForm.calendarEventColor,
            isRecurring: this.calendarEventForm.isRecurring,

            dateTimeStarted: moment(startDate).format(`YYYY-MM-DDT${moment(this.calendarEventForm.startTime).format('HH:mm:ss')}`),
            dateTimeEnded: moment(endDate).format(`YYYY-MM-DDT${moment(this.calendarEventForm.endTime).format('HH:mm:ss')}`),

            calendarEventGroupId: calendarEventGroupId,
          })
        }

        this.loading.calendarEventSubmitBtn = true
        try {
          const { data, error } = await supabase
            .from('CalendarEvent')
            .insert(calendarEvents)

          if (error) {
            ElMessage.error('Failed to add Calendar Event')
            return
          }

          ElMessage.success('Calendar Event added successfully')
          this.clear()
          this.getCalendarEventsByCalendarId(this.selectedCalendarId)
        }
        catch(error) {
          ElMessage.error('An unexpected error occurred')
          console.error(error)
        }
        finally{
          this.loading.calendarEventSubmitBtn = false
        }
      }

      /* EDIT NON-RECURRING */
      if(this.dialog.title === 'Edit Non-Recurring Event') {
        
      }
    },

    /* GET CALENDARS */
    async getCalendars() {
      this.loading.selectCalendarInput = true
      try{
        const { data, error } = await supabase
          .from('Calendar')
          .select('*')
          .order('dateTimeCreated', { ascending: false })

        this.calendars = data
      }
      catch(error) {
        ElMessage.error('An unexpected error occurred')
        console.error(error)
      }
      finally{
        this.loading.selectCalendarInput = false
      }
    },

    /* GET CALENDAR EVENTS BY CALENDAR ID */
    async getCalendarEventsByCalendarId(calendarId){
      this.selectedCalendarId = calendarId
    
      if(this.selectedCalendarId == undefined || this.selectedCalendarId == null || this.selectedCalendarId == '') {
        ElMessage.warning('Please select a calendar first')
        return
      }

      const loading = ElLoading.service({
        lock: true,
        text: 'Loading',
        background: 'rgba(0, 0, 0, 0.7)',
      })

      try{
        const { data, error } = await supabase
          .from('CalendarEvent')
          .select('*')
          .eq('calendarId', this.selectedCalendarId)
          .gte('dateTimeStarted', this.firstDayOfMonth)
          .lte('dateTimeEnded', this.lastDayOfMonth)

        /* SIDEBAR CHECK BOX EVENTS */
        this.calendarEvents = data.filter((value, index, self) =>
            index === self.findIndex((event) => event.calendarEventGroupId === value.calendarEventGroupId),
        )

        this.calendarOptions.events = data.map(event => ({
          title: event.calendarEventName,
          start: event.dateTimeStarted,
          end: moment(event.dateTimeEnded).format(),
          color: event.calendarEventColor,
          extendedProps: {
            calendarEventId: event.calendarEventId,
            calendarId: event.calendarId,
            calendarEventName: event.calendarEventName,
            calendarEventDescription: event.calendarEventDescription,
            calendarEventColor: event.calendarEventColor,
            isRecurring: event.isRecurring,
            calendarEventGroupId: event.calendarEventGroupId,
            startTime: moment(event.dateTimeStarted).format('HH:mm:ss'),
            endTime: moment(event.dateTimeEnded).format('HH:mm:ss'),
            dateTimeStarted: moment(event.dateTimeStarted).format(),
            dateTimeEnded: moment(event.dateTimeEnded).format(),
          },
          allDay:
            this.weekClicked || this.dayClicked
              ? false
              : event.isRecurring
                ? true
                : false,
          rrule:
            event.isRecurring ? {
                  freq: 'weekly',
                  interval: 5,
                  dtstart: event.dateTimeStarted,
                  until: event.dateTimeEnded,
                }
              : undefined,
          display: event.isRecurring ? 'list-item' : 'block',
          
        }))
      }
      catch(error) {
        ElMessage.error('An unexpected error occurred')
        console.error(error)
      }
      finally {
        loading.close()
      }
    },


    handleDateRange(info){
      if (info.start < new Date().setHours(0,0,0,0)) {
        ElMessage.warning('Cannot create event in the past')
        return false;
      }
      this.formController('Create Event')

      this.calendarEventForm.dateRange.start = moment(info.startStr).format()
      this.calendarEventForm.dateRange.end = moment(info.endStr).subtract(1, 'days').format()

      this.disableRecurringDays(moment(this.calendarEventForm.dateRange.start).format(), moment(this.calendarEventForm.dateRange.end).add(1, 'days').format())
    },

    async moveResizeEvent(info){console.log(info)},

    eventClick(info) {
      this.calendarEventForm.dateTimeStartedDetails = moment(info.event.extendedProps.dateTimeStarted).format('MM/DD/YYYY HH:mm')
      this.calendarEventForm.dateTimeEndedDetails = moment(info.event.extendedProps.dateTimeEnded).format('MM/DD/YYYY HH:mm')

      this.calendarEventForm.calendarEventId = info.event.extendedProps.calendarEventId
      this.calendarEventForm.calendarEventName = info.event.extendedProps.calendarEventName
      this.calendarEventForm.calendarEventDescription = info.event.extendedProps.calendarEventDescription
      this.calendarEventForm.calendarEventColor = info.event.extendedProps.calendarEventColor
      this.calendarEventForm.startTime = info.event.extendedProps.dateTimeStarted
      this.calendarEventForm.endTime = info.event.extendedProps.dateTimeEnded
      this.calendarEventForm.dateRange = { 
        start: moment(info.event.extendedProps.dateTimeStarted).format(),
        end:  moment(info.event.extendedProps.dateTimeEnded) .format()
        }
      this.calendarEventForm.isRecurring = info.event.extendedProps.isRecurring
      this.calendarEventForm.calendarId = info.event.extendedProps.calendarId
      this.calendarEventForm.calendarEventGroupId = info.event.extendedProps.calendarEventGroupId
      
      // NON-RECURRING
      if (info.event.extendedProps.isRecurring == false) {
        this.formController('Event Details')
       
      }
      // RECURRING
      else {
        this.formController('Open Recurring Event')
      }
    },

    /* DELETE CALENDAR */
    async deleteCalendarEvent() {
        await ElMessageBox.confirm(
          'Do you want to delete this Event?', 
          'Warning', 
          {
            confirmButtonText: 'OK', 
            cancelButtonText: 'Cancel', 
            type: 'warning', 
          }
        )

        const loading = ElLoading.service({
          lock: true,
          text: 'Loading',
          background: 'rgba(0, 0, 0, 0.7)',
        })

        try {
          /* RECURRING CALENDAR EVENT */
          if (this.calendarEventForm.isRecurring) {
            await supabase
              .from('CalendarEvent')
              .delete()
              .eq('calendarEventGroupId', this.calendarEventForm.calendarEventGroupId)
          } 
          /* NON-RECURRING CALENDAR EVENT */
          else {
            await supabase
              .from('CalendarEvent')
              .delete()
              .eq('calendarEventId', this.calendarEventForm.calendarEventId)
          }

          ElMessage.success("Event deleted successfully")
          this.clear()
          await this.getCalendarEventsByCalendarId(this.selectedCalendarId)

        } catch(error) {
          ElMessage.error('An unexpected error occurred')
          console.error(error)
        } finally {
          loading.close()
        }
    },

    /* CLEAR ALL */
    clear(){
      this.dialog.calendarForm = false
      this.dialog.calendarEventForm = false
      this.dialog.calendarEventConfirmation = false
      this.dialog.recurringEventOperation = false
      
      this.loading.calendarSubmitBtn = false
      this.loading.calendarEventSubmitBtn = false
      this.loading.selectCalendarInput = false

      this.calendarForm.calendarName = ''

      this.calendarEventForm.calendarEventId = ''
      this.calendarEventForm.calendarId = ''
      this.calendarEventForm.calendarEventName = ''
      this.calendarEventForm.calendarEventDescription = ''
      this.calendarEventForm.calendarEventColor = '#409EFF'
      this.calendarEventForm.isRecurring = false
      this.calendarEventForm.dateRange = {
        start: null,
        end: null
      }
      this.calendarEventForm.selectedRecurringDays = []

      this.operationForm.isSeries = false

    },

    // CHECK ALL CALENDAR EVENTS
    checkAllEvents() {
      this.calendarEvents.forEach((calendarEvent) => {
        this.selectedCalendarEvent[calendarEvent.calendarEventId] = true
      })

      this.calendarOptions.events = this.calendarOptions.events.map((event) => {
        return {
          ...event,
          display: (event.rrule ? 'list-item' : 'block') ?? 'none',
          backgroundColor: event.backgroundColor,
        }
      })
    },

    // UNCHECK ALL CALENDAR EVENTS
    uncheckAllEvents() {
      this.calendarEvents.forEach((calendarEvent) => {
        this.selectedCalendarEvent[calendarEvent.calendarEventId] = false
      })

      this.calendarOptions.events = this.calendarOptions.events.map((event) => {
        return {
          ...event,
          display: 'none',
          backgroundColor: event.backgroundColor,
        }
      })
    },

    // CHECK SHOW/HIDE INDIVIDUAL CALENDAR EVENT
    handleCheckboxChange(calendarEventId, calendarEventGroupId) {
      const isChecked = this.selectedCalendarEvent[calendarEventId]

      this.calendarOptions.events = this.calendarOptions.events.map((event) => {
        if (event.extendedProps.calendarEventGroupId === calendarEventGroupId) {
          console.log(calendarEventId)

          const displayStatus = isChecked ? (event.rrule ? 'list-item' : 'block') : 'none'

          return {
            ...event,
            display: displayStatus,
            backgroundColor: event.backgroundColor,
          }
        }
        return event
      })
    },

    /* CALENDAR EVENT FORM CHECK RECURRING DAYS */
    selectedRecurringDay(e) {
      const index = this.calendarEventForm.selectedRecurringDays.indexOf(e)
      if (index !== -1) {
        this.calendarEventForm.selectedRecurringDays.splice(index, 1)
      } else {
        this.calendarEventForm.selectedRecurringDays.push(e)
      }
      console.log(this.calendarEventForm.selectedRecurringDays)
    },

    /* CALENDAR EVENT FORM DISABLE RECURRING DAYS WHEN OUT OF BOUNDS */
    disableRecurringDays(startDate, endDate) {
      const start = moment(startDate)
      const end = moment(endDate)

      const daysInRange = new Set()
      let currentDate = start.clone()

      while (currentDate <= end) {
        daysInRange.add(currentDate.day())
        currentDate.add(1, 'day')
      }

      this.recurringDays = this.recurringDays.map(day => ({
        ...day,
        disabled: !daysInRange.has(day.value),
      }))

      // remove selected days that are now disabled
      this.calendarEventForm.selectedRecurringDays =
        this.calendarEventForm.selectedRecurringDays.filter(d =>
          daysInRange.has(d)
        )
    },

    /* SIDEBAR VCALENDAR CHECK MOVEMENT */
    didMove(info) {
      let formatDate = moment(info[0].id).add(1, 'days').format()
      this.calendarApi.gotoDate(formatDate)
      this.getCalendarEventsByCalendarId(this.selectedCalendarId)
    },

    /* SIDEBAR VCALENDAR DAY CLICK */
    dayClick(info) {
      this.weekClicked = false
      this.dayClicked = true
      this.calendarApi.changeView('timeGridDay')
      this.calendarApi.gotoDate(info.endDate)
      this.getCalendarEventsByCalendarId(this.selectedCalendarId)
    },
  },
  mounted(){
    this.calendarApi = this.$refs.refCalendar.getApi()
  },
  watch: {
    'calendarEventForm.dateRange': {
      handler(newRange) {
        if (!newRange?.start || !newRange?.end) return

        this.disableRecurringDays(newRange.start, newRange.end)
      },
      deep: true,
      immediate: true
    }
  }
}
</script>