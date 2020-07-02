<template>
  <fragment>
    <v-tooltip bottom>
      <template v-slot:activator="{ on }">
        <v-icon
          v-on="on"
          @click="openPlay()"
        >
          mdi-play-circle
        </v-icon>
      </template>
      <span>Play</span>
    </v-tooltip>
    <v-dialog
      v-model="dialog"
      max-width="1024px"
    >
      <v-card>
        <v-toolbar
          dark
          color="primary"
        >
          <v-toolbar-title>Play</v-toolbar-title>
          <v-spacer />
        </v-toolbar>
        <div ref="playterminal" />
        <v-card>
          <v-card-actions>
            <v-btn
              v-show="!disable"
              :disabled="disable"
              color="primary"
              class="mt-4"
              @click="connect()"
            >
              Play
            </v-btn>
            <v-slider
              v-model="currentTime"
              readonly
              min="0"
              :max="totalLength"
              :label="`${now} - ${end}`"
            />
          </v-card-actions>
        </v-card>
      </v-card>
    </v-dialog>
  </fragment>
</template>

<script>

import { Terminal } from 'xterm';
import { FitAddon } from 'xterm-addon-fit';
import moment from 'moment';
import 'xterm/css/xterm.css';

export default {
  name: 'SessionPlay',

  props: {
    logs: {
      type: Array,
      required: true,
    },
  },

  data() {
    return {
      dialog: false,
      disable: false,
      currentTime: 0,
    };
  },

  computed: {
    logsData: {
      get() {
        return this.logs;
      },
    },

    length() {
      return this.logs.length;
    },

    now() {
      return this.duration(this.currentTime).display;
    },

    end() {
      return this.duration(null).display;
    },

    totalLength() {
      return this.duration(null).intervalLength;
    },

  },

  watch: {
    dialog(value) {
      if (!value) {
        this.close();
      }
    },
  },

  methods: {
    openPlay() {
      this.dialog = !this.dialog;
      this.xterm = new Terminal({ // instantiate
        cursorBlink: true,
        fontFamily: 'monospace',
      });
      this.fitAddon = new FitAddon(); // load fit
      this.xterm.loadAddon(this.fitAddon); // adjust screen in container
    },

    duration(timeMs) {
      let interval = 0;
      if (!timeMs) { // not params, use default to maxlength
        const max = new Date(this.logsData[this.length - 1].time);
        const min = new Date(this.logsData[0].time);
        interval = max - min;
      } else {
        interval = timeMs;
      }
      const duration = moment.duration(interval, 'milliseconds');
      const tD = {
        seconds: Math.floor(duration.asSeconds()),
        minutes: Math.floor(duration.asMinutes()),
        hours: Math.floor(duration.asHours()),
      };
      return {
        display: `${tD.hours}:${tD.minutes}:${tD.seconds}`,
        intervalLength: interval,
      };
    },

    timer() {
      if (this.currentTime >= this.totalLength) return;
      this.currentTime += 100;
      this.iterativeTimer = setTimeout(this.timer.bind(null), 100);
    },

    connect() {
      this.disable = true;
      this.xterm.open(this.$refs.playterminal);
      this.$nextTick(() => this.fitAddon.fit());
      this.fitAddon.fit();
      this.xterm.focus();
      this.print(0, this.logsData);
      this.timer();
      if (this.xterm.element) { // check already existence
        this.xterm.reset();
      }
    },

    close() {
      if (this.xterm) this.xterm.dispose();
      this.disable = false;
      clearInterval(this.iterativePrinting);
      clearInterval(this.iterativeTimer);
      this.currentTime = 0;
    },

    print(i, obj) {
      this.xterm.write(`${this.logs[i].message}`);
      if (i === this.logs.length - 1) {
        return;
      }
      const nextObj = this.logs[i + 1];
      const now = new Date(obj.time);
      const future = new Date(nextObj.time);
      this.iterativePrinting = setTimeout(this.print.bind(null, i + 1, nextObj), future - now);
    },
  },
};

</script>
