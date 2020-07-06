<template>
  <fragment>
    <div
      v-if="notificationStatus"
    >
      <v-btn
        x-small
        color="primary"
        @click="setMessages(1)"
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
        @click="setMessages(1)"
      >
        Accept
      </v-btn>

      <v-btn
        v-if="operationType == 1"
        small
        outlined
        @click="setMessages(2)"
      >
        Reject
      </v-btn>

      <v-btn
        v-else-if="operationType == 3"
        small
        outlined
        @click="setMessages(3)"
      >
        Remove
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
            v-if="typeMessage == 1"
          >
            You are about to accept this device
          </div>

          <div
            v-else-if="typeMessage == 2"
          >
            You are about to reject this device
          </div>

          <div
            v-else-if="typeMessage == 3"
          >
            You are about to delete this device
          </div>
        </v-card-text>

        <v-card-actions>
          <v-spacer />

          <v-btn
            text
            @click="dialog=!dialog"
          >
            Cancel
          </v-btn>

          <v-btn
            v-if="typeMessage == 1"
            text
            @click="acceptDevice();"
          >
            Accept
          </v-btn>

          <v-btn
            v-if="typeMessage == 2"
            text
            @click="rejectDevice();"
          >
            Reject
          </v-btn>

          <v-btn
            v-if="typeMessage == 3"
            text
            @click="removeDevice();"
          >
            Remove
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

    notificationStatus: {
      type: Boolean,
      required: false,
      default: false,
    },

    // Operation Type Received
    // 1 - Accept <- pending
    // 2 - Reject <- deny
    // 3 - Remove <- unsed
    operationType: {
      type: Number,
      required: true,
    },
  },

  data() {
    return {
      dialog: false,
      operationAccept: false,
      typeMessage: 0,
    };
  },

  methods: {
    setMessages(operation) {
      this.dialog = !this.dialog;
      this.typeMessage = operation;
    },

    async acceptDevice() {
      await this.$store.dispatch('devices/accept', this.uid);
      this.refreshDevices();
    },

    async rejectDevice() {
      await this.$store.dispatch('devices/reject', this.uid);
      this.refreshDevices();
    },

    async removeDevice() {
      await this.$store.dispatch('devices/remove', this.uid);
      this.refreshDevices();
    },

    refreshDevices() {
      this.$emit('update');
      if (window.location.pathname === '/devices/pending' || window.location.pathname === '/devices') {
        this.$store.dispatch('devices/refresh');
        this.$store.dispatch('notifications/fetch');
      }

      this.dialog = !this.dialog;
    },
  },
};

</script>
