<template>
  <fragment>
    <div class="d-flex pa-0 align-center">
      <h1>Devices</h1>
      <v-spacer />
      <v-text-field
        v-model="search"
        append-icon="mdi-magnify"
        label="Search by hostname"
        class="mx-6"
        single-line
        hide-details
      />
      <v-spacer />
      <DeviceAdd />
      <v-btn
        class="mr-2"
        outlined
        @click="$store.dispatch('modals/showAddDevice', true)"
      >
        Add Device
      </v-btn>
    </div>
    <v-card class="mt-2">
      <v-app-bar
        flat
        color="transparent"
      >
        <v-tabs>
          <v-tab
            to="/devices"
          >
            Device List
          </v-tab>
          <v-tab
            to="/devices/pending"
          >
            Pending
          </v-tab>
          <v-tab
            to="/devices/rejected"
          >
            Rejected
          </v-tab>
        </v-tabs>
      </v-app-bar>
      <v-divider />

      <router-view />

      <v-snackbar
        v-model="copySnack"
        :timeout="3000"
      >
        Device SSHID copied to clipboard
      </v-snackbar>
    </v-card>
  </fragment>
</template>

<script>

import DeviceAdd from '@/components/device/DeviceAdd';

export default {
  name: 'DeviceList',

  components: {
    DeviceAdd,
  },

  data() {
    return {
      copySnack: false,
      search: '',
    };
  },

  watch: {
    pagination: {
      handler() {
        this.getDevices();
      },
      deep: true,
    },

    search() {
      this.getDevices();
    },
  },

  methods: {
    async getDevices() {
      await this.$store.dispatch('devices/setFilter', this.search);
      // eslint-disable-next-line no-console
      console.log(this.search);
      // this.$store.dispatch('devices/refresh2');
    },

    showCopySnack() {
      this.copySnack = true;
    },
  },
};

</script>

<style scoped>

.list-itens {
  font-family: monospace;
}

.icons{
  margin-right: 4px;
}

</style>
