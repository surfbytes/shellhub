<template>
  <fragment>
    <div
      v-if="!stateList"
    >
      <v-btn
        x-small
        color="primary"
        @click="statusOperation(true)"
      >
        Accept
      </v-btn>
    </div>

    <div
      v-else
    >
      <v-btn
        class="mr-2"
        small
        outlined
        @click="statusOperation(true)"
      >
        Accept
      </v-btn>
      <v-btn
        small
        outlined
        @click="statusOperation(false)"
      >
        Reject
      </v-btn>
    </div>

    <v-dialog
      v-model="dialog"
      max-width="400"
    >
      <v-card>
        <v-card-title class="headline grey lighten-2 text-center">
          Are you sure?
        </v-card-title>

        <v-card-text class="mt-4 mb-3 pb-1">
          <div
            v-if="operationAccept"
          >
            You are about to accept this device
          </div>
          <div
            v-else
          >
            You are about to reject this device
          </div>
        </v-card-text>

        <v-card-actions>
          <v-spacer />

          <v-btn
            text
            @click="dialog=!dialog"
          >
            Close
          </v-btn>

          <v-btn
            text
            @click="operationDevice();"
          >
            <div
              v-if="operationAccept"
            >
              Accept
            </div>
            <div
              v-else
            >
              Reject
            </div>
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </fragment>
</template>

<script>

export default {
  name: 'DeviceAccept',

  props: {
    uid: {
      type: String,
      required: true,
    },

    stateList: {
      type: Boolean,
      required: false,
      default: false,
    },
  },

  data() {
    return {
      dialog: false,
      operationAccept: false,
    };
  },

  methods: {
    statusOperation(operation) {
      this.operationAccept = operation;
      this.dialog = !this.dialog;
    },

    async operationDevice() {
      if (this.operationAccept) {
        await this.$store.dispatch('devices/accept', this.uid);
      } else {
        await this.$store.dispatch('devices/reject', this.uid);
      }

      this.$emit('update');
      if (window.location.pathname === '/devices/pending') {
        await this.$store.dispatch('devices/refresh');
        this.$store.dispatch('notifications/fetch');
      }

      this.dialog = !this.dialog;
    },
  },
};

</script>
