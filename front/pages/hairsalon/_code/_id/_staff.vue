<template>
    <v-app class="reserve-font-size">
        <!-- Header -->
        <v-app-bar app class="elevation-1" style="width:100%;" color="grey lighten-5">
            <v-toolbar-title>
                <v-select
                    solo
                    dense
                    prepend-icon="mdi-calendar"
                    v-bind:items="months"
                    v-model="selectedMonth"
                    class="mt-6"
                    style="max-width:185px;"
                    v-on:change="changeMonth"
                ></v-select>
            </v-toolbar-title>
            <v-toolbar-title class="ml-3">
                <div v-if="staff.sex=='male'">
                    <v-chip outlined color="blue lighten-1">
                        {{ staff.name }}
                        <v-avatar right class="hidden-xs-only">
                            <v-img v-bind:src="staff.img" />
                        </v-avatar>
                    </v-chip>
                </div>
                <div v-else-if="staff.sex=='female'">
                    <v-chip outlined color="pink lighten-1">
                        {{ staff.name }}
                        <v-avatar right class="hidden-xs-only">
                            <v-img v-bind:src="staff.img" />
                        </v-avatar>
                    </v-chip>
                </div>
                <div v-else>
                    <v-chip outlined color="deep-purple lighten-1">{{ staff.name }}</v-chip>
                </div>
            </v-toolbar-title>
            <v-spacer></v-spacer>
            <v-toolbar-title>
                <v-chip outlined class="hidden-xs-only float-right">
                    <span class="info-hairsalon line-color">{{ area ? area.name : null }}&nbsp;&nbsp;{{ hairsalon ? hairsalon.name : null }}</span>
                </v-chip>
            </v-toolbar-title>
        </v-app-bar>
        <!-- Weekdays Headeră€€-->
        <v-container fluid style="margin-top:82px;">
            <v-row justify="center">
                <v-col class="ma-0 pa-0 text-center"><span style="font-weight:bold; color:red;">{{ $t("utils.sun") }}</span></v-col>
                <v-col class="ma-0 pa-0 text-center"><span style="font-weight:bold;">{{ $t("utils.mon") }}</span></v-col>
                <v-col class="ma-0 pa-0 text-center"><span style="font-weight:bold;">{{ $t("utils.tue") }}</span></v-col>
                <v-col class="ma-0 pa-0 text-center"><span style="font-weight:bold;">{{ $t("utils.wed") }}</span></v-col>
                <v-col class="ma-0 pa-0 text-center"><span style="font-weight:bold;">{{ $t("utils.thu") }}</span></v-col>
                <v-col class="ma-0 pa-0 text-center"><span style="font-weight:bold;">{{ $t("utils.fri") }}</span></v-col>
                <v-col class="ma-0 pa-0 text-center"><span style="font-weight:bold; color:blue;">{{ $t("utils.sat") }}</span></v-col>
            </v-row>
        </v-container>
        <!-- Calendar -->
        <v-sheet width="100%" height="calc(100% - 151px)">
            <v-calendar
                v-bind:locale="locale"
                v-bind:month-format="monthFormat"
                v-bind:day-format="dayFormat"
                v-bind:weekday-format="weekdayFormat"
                v-bind:interval-format="intervalFormat"
                v-model="calendar"
                class="mt-0 pt-2"
            >
                <template v-slot:day="context">
                    <div v-if="context.date.replace('-', '').substr(0,6) == selectedMonth" style="height:100%;">
                        <div v-if="isValid(context.date, minDate, maxDate)" v-bind:style="dayStyle(context.date, selectedMonth, minDate, maxDate)">
                            <v-hover v-slot:default="{ hover }" open-delay="150">
                                <v-card color="white" class="daily-card" v-ripple="{ class: 'black--text' }" v-bind:elevation="hover ? 16 : 2" v-on:click="showDayDetail(context.date)">
                                    <v-card-text class="ma-0 pa-0 pt-lg-3 pt-1" style="font-size:1.0em; color:black; white-space:nowrap;">
                                        <div>{{ dateFormat(context.date, selectedMonth, $vuetify.breakpoint) }}</div>
                                        <div v-if="dayStatus(context.date)==1">
                                            <div class="pa-1 hidden-xs-only daily-height">
                                                <div>{{ $t("calendar.reservation") }}</div>
                                                <v-btn small rounded color="success" class="ma-0 pa-0 btn-opacity">{{ $t("calendar.yes") }}</v-btn>
                                            </div>
                                            <div class="hidden-sm-and-up text-center daily-height-small">
                                                <v-btn small fab color="success" class="ma-0 pa-0 btn-opacity">{{ $t("calendar.short_yes") }}</v-btn>
                                            </div>
                                        </div>
                                        <div v-else-if="dayStatus(context.date)==2">
                                            <div class="pa-1 hidden-xs-only daily-height">
                                                <div>{{ $t("calendar.reservation") }}</div>
                                                <v-btn small rounded color="error" class="ma-0 pa-0 btn-opacity">{{ $t("calendar.no") }}</v-btn>
                                            </div>
                                            <div class="hidden-sm-and-up text-center daily-height-small">
                                                <v-btn small fab color="error" class="ma-0 pa-0 btn-opacity">{{ $t("calendar.short_no") }}</v-btn>
                                            </div>
                                        </div>
                                        <div v-else>
                                            <div class="hidden-xs-only daily-height">
                                                <div class="ma-2 font-weight-bold" style="color:red;">{{ $t("calendar.closingday") }}</div>
                                            </div>
                                            <div class="hidden-sm-and-up daily-height-small">
                                                <div class="ma-2 font-weight-bold" style="color:red;">{{ $t("calendar.short_closingday") }}</div>
                                            </div>
                                        </div>
                                    </v-card-text>
                                </v-card>
                            </v-hover>
                        </div>
                        <div v-else style="width:100%; height:100%; background-color:#f8f8ff;">
                            <div class="hidden-sm-and-up" style="min-height:70px;"></div>
                        </div>
                    </div>
                    <div v-else style="width:100%; height:100%; background-color:#dcdcdc;">
                    </div>
                </template>
            </v-calendar>
        </v-sheet>

        <!-- Detail Dialog -->
        <v-dialog v-model="dialog" fullscreen hide-overlay transition="dialog-bottom-transition">
            <v-card>
                <v-toolbar color="grey lighten-5">
                    <v-btn icon v-on:click="dialog=false">
                        <v-icon>mdi-close</v-icon>
                    </v-btn>
                    <v-toolbar-title>
                        <span class="daily-title">{{ reserveDate }} {{ hairsalon.name }}&nbsp;&nbsp;{{ $t("calendar.msg001") }}</span>
                    </v-toolbar-title>
                </v-toolbar>
                <div class="text-center ma-2">
                    <div v-if="staff.sex=='male'">
                        <v-alert dense outlined color="blue lighten-1" style="color:white; opacity:0.9; margin-bottom:0;">
                            <span>{{ staff.name }}</span>
                            <v-avatar size="30">
                                <v-img v-bind:src="staff.img" />
                            </v-avatar>
                            <span>{{ courseName }}</span>
                        </v-alert>
                    </div>
                    <div v-else-if="staff.sex=='female'">
                        <v-alert dense outlined color="pink lighten-1" style="color:white; opacity:0.9; margin-bottom:0;">
                            <span>{{ staff.name }}</span>
                            <v-avatar size="30">
                                <v-img v-bind:src="staff.img" />
                            </v-avatar>
                            <span>{{ courseName }}</span>
                        </v-alert>
                    </div>
                    <div v-else>
                        <v-alert dense outlined color="deep-purple lighten-1" style="color:white; opacity:0.9; margin-bottom:0;">
                            <span>{{ staff.name }}</span>
                            <span style="margin-left:12px;"></span>
                            <span>{{ courseName }}</span>
                        </v-alert>
                    </div>
                </div>
                <v-calendar 
                    v-bind:locale="locale"
                    type="day"
                    event-overlap-mode="column"
                    v-model="reserveDate"
                    v-bind:events="events"
                    v-bind:event-color="eventColor"
                    v-bind:event-name="eventNameFormat"
                    v-bind:event-timed="eventTimedFormat"
                    v-bind:first-interval="8"
                    v-on:click:time="clickTime"
                    v-on:click:event="clickEvent"
                >
                    <template v-slot:interval="props">
                        <div v-bind:style="intervalStyle(props)"></div>
                    </template>
                </v-calendar>
            </v-card>
            <v-btn 
                fixed
                rounded
                absolute bottom right
                color="success"
                class="mb-10"
                style="width:160px; color:white; opacity:0.9;"
                v-on:click="showDialog"
                v-bind:disabled="reserveButton"
            >
                {{ $t("calendar.msg002") }}
            </v-btn>
            <!-- Reserve Dialog -->
            <v-dialog persistent max-width="460px" v-model="reserveDialog">
                <v-toolbar dense class="elevation-0">
                    <v-btn icon style="position:absolute;" v-on:click="closeDialog">
                        <v-icon>mdi-close</v-icon>
                    </v-btn>
                    <div style="margin:auto; font-size:1.0em;">{{ $t("calendar.msg003") }}</div>
                </v-toolbar>
                <v-card class="elevation-0" style="border-radius:0;">
                    <v-card-text class="pb-0">
                        <v-form ref="form" v-model="valid" lazy-validation>
                            <v-container>
                                <v-row dense>
                                    <v-col cols="12">
                                        <v-select
                                            v-bind:label="$t('calendar.msg004')"
                                            v-bind:items="times"
                                            item-text="text"
                                            item-value="value"
                                            v-model="sendReserve.start"
                                            required
                                            v-bind:rules="[v=>!!v || $t('calendar.msg005')]"
                                            v-on:change="calcTime(sendReserve.start, sendReserve.menu)"
                                            v-bind:readonly="reserveDialogReadonly"
                                        ></v-select>
                                    </v-col>
                                </v-row>
                                <v-row dense>
                                    <v-col cols="12">
                                        <v-select
                                            v-bind:label="$t('calendar.msg006')"
                                            v-bind:items="menu"
                                            item-text="text"
                                            return-object
                                            required
                                            v-bind:rules="[v=>!!v || $t('calendar.msg007')]"
                                            v-model="sendReserve.menu"
                                            v-on:change="calcTime(sendReserve.start, sendReserve.menu)"
                                            v-bind:readonly="true || reserveDialogReadonly"
                                        ></v-select>
                                    </v-col>
                                </v-row>
                                <v-row dense>
                                    <v-col cols="12">
                                        <v-select
                                            v-bind:label="$t('calendar.msg008')"
                                            v-bind:items="times"
                                            item-text="text"
                                            item-value="value"
                                            readonly
                                            required
                                            v-model="sendReserve.end"
                                            placeholder=" "
                                            class="font-weight-bold"
                                        ></v-select>
                                    </v-col>
                                    <div class="coupon-notice pt-1">
                                        <span v-bind:style="noteStyle" style="float:left;">&nbsp;&nbsp;â€»{{ $t("calendar.msg009") }}</span>
                                    </div>                                
                                </v-row>
                            </v-container>
                        </v-form>
                    </v-card-text>
                </v-card>
                <footer style="text-align:center; width:100%;">
                    <v-btn 
                        class="ma-0 font-weight-bold"
                        style="width:100%; height:48px; background-color:#00ba00; border-radius:0 0 4px 4px;"
                        v-on:click="reserve(sendReserve)"
                    >
                        <span style="color:#fff;">{{ $t("calendar.msg010") }}</span>
                    </v-btn>
                </footer>
            </v-dialog>
            <!-- Footer -->
            <vue-footer icons="4" v-bind:shop="hairsalon.id" v-bind:staff="staff"></vue-footer>
        </v-dialog>

        <!-- Error Message Dialog -->
        <v-dialog v-model="errorDialog" max-width="300px">
            <v-card>
                <v-card-title class="line-background-color">
                    <v-btn absolute icon style="right:0; top:-5px;" v-on:click="errorDialog=false">
                        <v-icon small>mdi-close</v-icon>
                    </v-btn>
                </v-card-title>
                <v-card-text class="text-center font-weight-bold pa-6 pb-3" style="color:red; line-height:1.4em; font-size:1.3em;">
                    <span v-html="errorDialogMessage.title"></span>
                </v-card-text>
                <v-card-text>
                    <span v-html="errorDialogMessage.text"></span>
                </v-card-text>
            </v-card>
        </v-dialog>
        <!-- Footer -->
        <vue-footer icons="3" v-bind:shop="hairsalon.id" v-bind:staff="staff"></vue-footer>
    </v-app>
</template>

<script>
/**
 * äºˆç´„ă‚«ăƒ¬ăƒ³ăƒ€ăƒ¼ç”»é?¢
 * 
 */
import VueFooter from "~/components/hairsalon/Footer.vue"

export default {
    layout: "reserve/hairsalon",
    components: {
        VueFooter,
    },
    async asyncData({ app, store, params, redirect }) {
        const months = app.$hairsalon.utils.monthList(2);
        const minDate = app.$utils.now("yyyymmdd");
        const maxDate = app.$utils.now("yyyymmdd", 2);
        // å‰?ç”»é?¢ă‚ˆă‚?å€¤å?–å¾—
        let area = app.$flash.hold("area");
        let hairsalon = app.$flash.hold("hairsalon");
        let staff = app.$flash.hold("staff");
        let selectedMenu = app.$flash.hold("menu");
        const targetDate = app.$flash.get("targetDate");
        // ăƒªăƒ­ăƒ¼ăƒ‰å¯¾å¿œ
        if (area === undefined) {
            const data = await app.$hairsalon.getAreaShops();
            area = data.areas.find((v) => v.code == params.code);
            hairsalon = data.hairsalons[params.code].find((v) => v.id == params.id);
        }
        if (staff === undefined) {
            const staffs = await app.$hairsalon.getShopStaffs(hairsalon.id);
            staff = staffs.find((v) => v.staffId == params.staff);
        }
        if (selectedMenu === undefined) {
            redirect({ path: `/hairsalon/${params.code}/${params.id}` });
            return;
        }

        // äºˆç´„æ™‚é–“å¸¯ăƒªă‚¹ăƒˆå?–å¾—
        const times = app.$hairsalon.utils.timeList(hairsalon.start, hairsalon.end);
        // ă‚³ăƒ¼ă‚¹ăƒªă‚¹ăƒˆå?–å¾—
        const menuPromise = app.$hairsalon.getCourses(hairsalon.id);
        // äºˆç´„ç?¶æ³?å?–å¾—
        const schedulePromise = app.$hairsalon.getStaffMonthlySchedule(hairsalon.id, staff.staffId, months[0].value, { shop: hairsalon, staff: staff, menu: selectedMenu });

        let menu = await menuPromise;
        let schedule = await schedulePromise;
        let sendReserve = {
                day: null,
                start: null,
                end: null,
                menu: selectedMenu,
        };

        return {
            schedule: schedule,
            area: area,
            hairsalon: hairsalon,
            staff: staff,
            selectedMonth: months[0].value,
            targetDate: targetDate,
            minDate: minDate,
            maxDate: maxDate,
            months: months,
            times: times,
            menu: menu,
            sendReserve: sendReserve,
            courseName: `${selectedMenu.name}ï¼ˆ${selectedMenu.time}åˆ†ï¼‰`,
        }
    },
    head() {
        return {
            title: this.$t("title")
        }
    },
    data() {
        return {
            reserveDate: null,
            schedule: null,
            area: null,
            hairsalon: null,
            staff: null,
            selectedMonth: null,
            targetDate: null,
            minDate: null,
            maxDate: null,
            months: null,
            courseName: null,
            dialog: false,
            reserveDialog: false,
            reserveButton: false,
            reserveDialogReadonly: false,
            dayEventClicked: false,
            events: [],
            times: null,
            course: null,
            valid: true,
            sendReserve: {
                day: null,
                start: null,
                end: null,
                menu: null,
            },
            errorDialog: false,
            errorDialogMessage: {
                title: null,
                text: null
            },
            noteStyle: {
                position: "relative",
                top: "-30px",
                fontSize: "0.7em",
                color: "red",
            },
        }
    },
    computed: {
        calendar() {
            let month = this.selectedMonth;
            if (month != null) {
                month = `${month.substr(0,4)}-${month.substr(4,2)}-01`;
            }
            return month;
        },
        locale() {
            let ret = "ja-jp"
            switch (this.$store.state.locale) {
            case "ja": ret = "ja-jp";break;
            case "en": ret = "en"; break;                
            }
            return ret;
        }
    },
    created() {
        this.$nuxt.$on("dialog", this.showDefaultDialog);
    },
    mounted() {
        this.$nextTick(() => {
            window.addEventListener("resize", this.modifyWeeksHeight);
            this.modifyWeeksHeight();
            if (this.targetDate) {
                this.showDayDetail(this.targetDate, 1);
            }
        });
    },
    updated() {
        this.$nextTick(() => {
            this.modifyWeeksHeight();
        });
    },
    beforeDestroy() {
        window.removeEventListener("resize", this.modifyWeeksHeight);
    },
    methods: {
        /**
         * ă‚«ăƒ¬ăƒ³ăƒ€ăƒ¼å¹´æœˆăƒ•ă‚©ăƒ¼ăƒ?ăƒƒăƒˆ
         * 
         * @param {string} datetime æ—¥ä»˜
         * @returns {string} å¹´æœˆ
         */
        monthFormat(datetime) {
            let date = new Date(datetime.date);
            let month = date.getMonth() + 1;
            return `${month} /`;
        },

        /**
         * ă‚«ăƒ¬ăƒ³ăƒ€ăƒ¼æ—¥ăƒ•ă‚©ăƒ¼ăƒ?ăƒƒăƒˆ
         * 
         * @param {Object} datetime v-calendaræ—¥ä»˜ă‚ªăƒ–ă‚¸ă‚§ă‚¯ăƒˆ
         * @returns {number} æ—¥
         */
        dayFormat(datetime) {
            return datetime.day;
        },

        /**
         * ă‚«ăƒ¬ăƒ³ăƒ€ăƒ¼æ›œæ—¥ăƒ•ă‚©ăƒ¼ăƒ?ăƒƒăƒˆ
         * 
         * @param {Object} datetime v-calendaræ—¥ä»˜ă‚ªăƒ–ă‚¸ă‚§ă‚¯ăƒˆ
         * @returns {number} æ›œæ—¥å€¤
         */
        weekdayFormat(datetime) {
            return null;
        },

        /**
         * ă‚«ăƒ¬ăƒ³ăƒ€ăƒ¼é–“é?”ăƒ•ă‚©ăƒ¼ăƒ?ăƒƒăƒˆ
         * 
         * @returns {number} é–“é?”å€¤
         */
        intervalFormat() {
            return null;
        },

        /**
         * ă‚«ăƒ¬ăƒ³ăƒ€ăƒ¼æœ‰å?¹æ—¥åˆ¤å®?
         * 
         * @param {string} date æ—¥ä»˜
         * @param {number} min æœ€å°?æ—¥ä»˜
         * @param {number} max æœ€å¤§æ—¥ä»˜
         * @returns {boolean} çœŸå?½å€¤
         */
        isValid(date, min, max) {
            let ret = false;

            const fmtdate = date.replace(/-/g, "");
            if (fmtdate >= min && fmtdate <= max) {
                ret = true;
            }

            return ret;
        },

        /**
         * ă‚«ăƒ¬ăƒ³ăƒ€ăƒ¼æ—¥ă‚¹ăƒ†ăƒ¼ă‚¿ă‚¹
         * 
         * @param {string} date æ—¥ä»˜
         * @returns {number} äºˆç´„ç?¶æ³?ă‚¹ăƒ†ăƒ¼ă‚¿ă‚¹
         */
        dayStatus(date) {
            let ret = 1;
            if (date in this.schedule) {
                ret = this.schedule[date].status;
            } else {
                ret = this.$hairsalon.utils.isHoliday(date, this.hairsalon.holiday) ? 0 : 1;
            }
            return ret;
        },

        /**
         * ă‚«ăƒ¬ăƒ³ăƒ€ăƒ¼æ—¥Styleå±?æ€§
         * 
         * @param {string} date æ—¥ä»˜
         * @param {string} month å¹´æœˆ
         * @param {string} min æœ€å°?æ—¥ä»˜
         * @param {string} max æœ€å¤§æ—¥ä»˜
         * @returns {Object} Styleå±?æ€§
         */
        dayStyle(date, month, min, max) {
            let style = {
                textAlign: "center",
                width: "99%",
                height: "99%",
                margin: "auto",
            };
            return style;
        },

        /**
         * æ—¥ă‚«ăƒ¬ăƒ³ăƒ€ăƒ¼é–“é?”Styleå±?æ€§
         * 
         * @param {Object} props v-calendaræ—¥ă‚ªăƒ–ă‚¸ă‚§ă‚¯ăƒˆ
         * @returns {Object} Styleå±?æ€§
         */
        intervalStyle(props) {
            let style = { width: "100%", height: "100%" };

            const time = props.time;
            if (this.hairsalon.start <= time && this.hairsalon.end > time) {
                style.backgroundColor = "gray";
                style.opacity = "0.2";
            }

            return style;
        },

        /**
         * ă‚«ăƒ¬ăƒ³ăƒ€ăƒ¼æ—¥ä»˜è¡¨ç¤ºăƒ•ă‚©ăƒ¼ăƒ?ăƒƒăƒˆ
         * 
         * @param {string} date æ—¥ä»˜
         * @param {string} month å¹´æœˆ
         * @param {Object} breakpoint Vuetifyăƒ–ăƒ¬ăƒ¼ă‚¯ăƒ?ă‚¤ăƒ³ăƒˆ
         * @returns {string} æœˆæ—¥
         */
        dateFormat(date, month, breakpoint) {
            let yyyymmdd = date.split("-");
            let mmdd = `${parseInt(yyyymmdd[1], 10)} / ${parseInt(yyyymmdd[2], 10)}`;
            if (breakpoint.xs) {
                mmdd = mmdd.replace(/\s/g, "");
            }
            return mmdd;
        },

        /**
         * ă‚«ăƒ¬ăƒ³ăƒ€ăƒ¼äºˆç´„ă‚¤ăƒ™ăƒ³ăƒˆè¡¨ç¤ºæ–‡è¨€ăƒ•ă‚©ăƒ¼ăƒ?ăƒƒăƒˆ
         * 
         * @param {Object}} event äºˆç´„ă‚¤ăƒ™ăƒ³ăƒˆ
         * @param {boolean} timed æ™‚é–“æŒ‡å®?ăƒ•ăƒ©ă‚°
         * @returns {string} HTMLæ–‡å­—åˆ—
         */
        eventNameFormat(event, timed) {
            const name = event.input.name;
            return this.eventText(name, event);
        },

        /**
         * ă‚«ăƒ¬ăƒ³ăƒ€ăƒ¼æ™‚é–“å¸¯ăƒ•ă‚©ăƒ¼ăƒ?ăƒƒăƒˆ
         * 
         * @param {Object} event äºˆç´„ă‚¤ăƒ™ăƒ³ăƒˆ
         */
        eventTimedFormat(event) {

        },

        /**
         * ă‚«ăƒ¬ăƒ³ăƒ€ăƒ¼ă‚¤ăƒ™ăƒ³ăƒˆè‰²
         * 
         * @param {Object} event äºˆç´„ă‚¤ăƒ™ăƒ³ăƒˆ
         * @returns {string} ă‚¤ăƒ™ăƒ³ăƒˆè‰²
         */
        eventColor(event) {
            return event.color;
        },

        /**
         * äºˆç´„ă‚¤ăƒ™ăƒ³ăƒˆè¡¨ç¤ºHTML
         * 
         * @param {string} text è¡¨ç¤ºăƒ†ă‚­ă‚¹ăƒˆ
         * @param {Object} event äºˆç´„ă‚¤ăƒ™ăƒ³ăƒˆ
         * @returns {string} HTMLæ–‡å­—åˆ—
         */
        eventText(text, event) {
            const fromto = event.input.start.split(" ")[1] + " - " + event.input.end.split(" ")[1];
            return `<div class='event-text' style='height:600px; padding:2px; font-size:1.1em;'>${text}ă€€${fromto}</div>`;            
        },

        /**
         * äºˆç´„çµ‚äº†æ™‚é–“ç®—å‡º
         * 
         * @param {string} start äºˆç´„é–‹å§‹æ™‚é–“
         * @param {Object} menu ăƒ¡ăƒ‹ăƒ¥ăƒ¼æƒ…å ±
         */
        calcTime(start, menu) {
            if (!start || !menu) { return; }

            let etime = start.split(":");
            const hour = Math.floor(menu.time / 60);
            const minute = menu.time % 60;

            etime[0] = parseInt(etime[0], 10) + hour;
            etime[1] = parseInt(etime[1], 10) + minute;
            if (parseInt(etime[1],10) >= 60) { 
                etime[0]++;
                etime[1] = 0;
            }

            etime[0] = ("00"+etime[0]).slice(-2);
            etime[1] = ("00"+etime[1]).slice(-2);
            const endTime = `${etime[0]}:${etime[1]}`;
            this.sendReserve.end = endTime;
        },

        /**
         * é€±è¡¨ç¤ºé«˜ă?•ä¿®æ­£å‡¦ç?†
         * 
         */
        modifyWeeksHeight() {
            const cards = document.getElementsByClassName("v-calendar-weekly__week");
            for (const card of cards) {
                if (card.clientHeight < 92) {
                    card.style.height = "92px";
                }
            }
        },

        /**
         * å¹´æœˆå¤‰æ›´æ™‚ă‚«ăƒ¬ăƒ³ăƒ€ăƒ¼è¡¨ç¤ºå‡¦ç?†
         * 
         * @param {string} month å¹´æœˆ
         */
        changeMonth(month) {
            const hairsalonId = this.hairsalon.id;
            const staffId = this.staff.staffId;
            const info = { shop: this.hairsalon, staff: this.staff, menu: this.sendReserve.menu };
            // æ‹…å½“è€…æœˆåˆ¥ äºˆç´„ç?¶æ³?å?–å¾—
            this.$hairsalon.getStaffMonthlySchedule(hairsalonId, staffId, month, info)
            .then((data) => {
                this.schedule = data;
            });
        },

        /**
         * æ—¥æ™‚é–“å¸¯ă‚«ăƒ¬ăƒ³ăƒ€ăƒ¼è¡¨ç¤ºå‡¦ç?†
         * 
         * @param {string} date æ—¥ä»˜
         */
        async showDayDetail(date) {
            const status = this.dayStatus(date);
            if (status==0 || status==2) { return; } // ă€Œå®?ä¼‘æ—¥ă€?ă?¨ă€Œä¸?å?¯ă€?ă?¯è©³ç´°è¡¨ç¤ºă?—ă?ªă?„

            let schedule = (date in this.schedule) ? this.schedule[date] : this.$hairsalon.utils.createScheduleRecord();
            this.reserveDate = date;
            this.reserveButton = (schedule.status==1) ? false : true;
            // æ—¥åˆ¥äºˆç´„ç?¶æ³?å?–å¾—
            const shopId = this.hairsalon.id;
            const staffId = this.staff.staffId;
            schedule.events = await this.$hairsalon.getStaffDailySchedule(shopId, staffId, date, schedule, this.hairsalon, this.menu, this.sendReserve.menu);

            this.events = schedule.events;
            this.sendReserve.day = date;
            this.dialog = true;
        },

        /**
         * æ—¥æ™‚é–“å¸¯ă‚«ăƒ¬ăƒ³ăƒ€ăƒ¼è¡¨ç¤ºå‡¦ç?†ï¼ˆåˆ?æœŸè¡¨ç¤ºç”¨ï¼‰
         * 
         * @param {boolean} opened è¡¨ç¤ºăƒ•ăƒ©ă‚°
         */
        showDefaultDialog(opened) {
            this.dialog = opened;
        },

        /**
         * äºˆç´„å†…å®¹å…¥å?›ăƒ€ă‚¤ă‚¢ăƒ­ă‚°è¡¨ç¤ºå‡¦ç?†
         * 
         * @param {string} start äºˆç´„é–‹å§‹æ™‚é–“
         * @param {string} end äºˆç´„çµ‚äº†æ™‚é–“
         * @param {number} course äºˆç´„ă‚³ăƒ¼ă‚¹ID
         */
        showDialog(start, end, course) {
            this.sendReserve.start = (typeof(start)=="string") ? start : null;
            this.sendReserve.end = (typeof(end)=="string") ? end : null;
            this.calcTime(this.sendReserve.start, this.sendReserve.menu);
            this.reserveDialog = true;
            setTimeout(()=>{
                this.$refs.form.resetValidation();
            }, 0);
        },

        /**
         * ăƒ€ă‚¤ă‚¢ăƒ­ă‚°é??è¡¨ç¤ºå‡¦ç?†
         * 
         */
        closeDialog() {
            this.reserveDialog = false;
            this.reserveDialogReadonly = false;
        },

        /**
         * äºˆç´„å†…å®¹å…¥å?›ăƒ€ă‚¤ă‚¢ăƒ­ă‚°è¡¨ç¤ºå‡¦ç?†ï¼ˆæ™‚é–“å¸¯ă‚¯ăƒªăƒƒă‚¯æ™‚ï¼‰
         * 
         * @param {Object} e ă‚¤ăƒ™ăƒ³ăƒˆă‚ªăƒ–ă‚¸ă‚§ă‚¯ăƒˆ
         */
        clickTime(e) {
            if (this.dayEventClicked) { 
                this.dayEventClicked = false;
                return;
            }

            const time = e.time.split(":");
            let start = `${time[0]}:00`;
            // å¯¾è±¡æ™‚é–“å¸¯äºˆç´„æ¸ˆă?¿ă?®å ´å?ˆă€?30åˆ†ă??ă‚‰ă?™
            for (const event of this.events) {
                let clickTime = `${this.sendReserve.day} ${start}`;
                if (event.start <= clickTime && clickTime < event.end) {
                    start = `${time[0]}:30`;       
                }
            }
            if (this.hairsalon.start <= start && this.hairsalon.end > start) {
                this.showDialog(start);
            }
        },

        /**
         * äºˆç´„ă?‚ă‚?ă‚¤ăƒ™ăƒ³ăƒˆă‚¯ăƒªăƒƒă‚¯æ™‚å‡¦ç?†
         * 
         * @param {Object} e ă‚¤ăƒ™ăƒ³ăƒˆă‚ªăƒ–ă‚¸ă‚§ă‚¯ăƒˆ
         */
        clickEvent(e) {
            this.dayEventClicked = true;
            const status = e.event.color;
            if (status == "error" || status == "warning") { return false; }

            this.reserveDialogReadonly = true;
            const start = e.event.start.split(" ")[1];
            const end = e.event.end.split(" ")[1];
            const course = e.event.course;
            this.showDialog(start, end, course);
        },

        /**
         * äºˆç´„å‡¦ç?†
         * 
         * @param {Object} input äºˆç´„å…¥å?›å†…å®¹
         * @returns {boolean} æ­£å¸¸ăƒ»ç•°å¸¸çµ‚äº†å€¤
         */
        async reserve(input) {
            const ret = this.$refs.form.validate();

            if (ret) {
                const token = this.$store.state.lineUser.token;
                const shopId = this.hairsalon.id;
                const day = input.day;
                const start = input.start;
                const end = input.end;
                const courseId = input.menu.id;
                const staffId = this.staff.staffId;
                const price = input.menu.price;
                const names = {
                    shopName: this.hairsalon.name,
                    courseName: input.menu.name,
                    staffName: this.staff.name,
                    userName: this.$store.state.lineUser.name
                };

                // å–¶æ¥­æ™‚é–“å¤–ăƒ?ă‚§ăƒƒă‚¯
                if (this.hairsalon.end.replace(" ", "") < end) {
                    this.errorDialogMessage.title = this.$t("calendar.msg011");
                    this.errorDialogMessage.text = this.$t("calendar.msg012");
                    this.errorDialog = true;
                    return false;
                }
                // æ™‚é–“é‡?è¤‡ăƒ?ă‚§ăƒƒă‚¯
                const overlaped = this.$hairsalon.utils.isTimezoneOverlap(this.events, day, start, end);
                if (overlaped) {
                    this.errorDialogMessage.title = this.$t("calendar.msg013");
                    this.errorDialogMessage.text = this.$t("calendar.msg014");
                    this.errorDialog = true;
                    return false;
                }

                try {
                    this.$processing.show(0, this.$t("calendar.msg015"));
                    // äºˆç´„ç™»éŒ²é€?ä¿¡
                    const data = await this.$hairsalon.updateReserve(token, shopId, day, start, end, courseId, staffId, price, names);
                    if (data) {
                        const reservationId = data.reservationId;
                        if (isNaN(reservationId)) {
                            this.reserveDialog = false;
                        } else {
                            switch (parseInt(reservationId, 10)) {
                            case -1:
                                this.errorDialogMessage.title = this.$t("calendar.msg016");
                                this.errorDialogMessage.text = this.$t("calendar.msg017");
                                break;
                            default:
                                this.errorDialogMessage.title = this.$t("calendar.msg018");
                                this.errorDialogMessage.text = this.$t("calendar.msg019");
                            }
                            this.errorDialog = true;
                            return false;
                        }
                        // ăƒ?ăƒ¼ă‚¸é?·ç§»
                        const message = {
                            name: this.$store.state.lineUser.name,
                            hairsalon: this.hairsalon,
                            staff: this.staff,
                            menu: input.menu,
                            day: input.day,
                            start: input.start,
                            end: input.end,
                        };
                        this.$flash.set("message", message);
                        this.$router.push("/hairsalon/completed");
                    }
                } finally {
                    this.$processing.hide();
                }

                return true;
            }
        },
    }
}
</script>

<style scoped>
.event-text {
    width: 100%;
    overflow: hidden;
    background-color: transparent;
}
.v-chip:before {
    background-color: transparent;
}
.reserve-font-size {
    font-size: 16px;
}
.daily-title {
    font-size: 1.0em;
}
.daily-card {
    height: 100%;
}
.btn-opacity {
    opacity: 0.8;
}
@media screen and (max-width:540px) {
    .reserve-font-size {
        font-size: 12px;
    }
    .daily-title {
        font-size: 0.8em;
    }
}
@media screen and (orientation:landscape) {
    .daily-height {
        margin-bottom: 7px;
    }
    .daily-height-small {
        margin-bottom: 12px;
    }
}
</style>
<style>
.v-calendar-weekly__day-label {
    display: none;
}
.v-event-timed.info.white--text {
    pointer-events: none;
    opacity: 0.0;
}
.v-event-timed.error.white--text {
    opacity: 0.8;
}
.v-event-timed.warning.white--text {
    opacity: 0.8;
}
.v-event-timed.orange.lighten-1.white--text {
    opacity: 0.8;
}
</style>
