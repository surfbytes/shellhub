<template>
  <fragment>
    <div class="d-flex pa-0 align-center">
      <h1>Pending Devices</h1>
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
    </div>
    <v-card class="mt-2">
      <v-app-bar
        flat
        color="transparent"
      />

      <v-divider />

      <v-card-text class="pa-0">
        <v-data-table
          class="elevation-1"
          :headers="headers"
          :items="getListPendingDevices"
          item-key="uid"
          :sort-by="['started_at']"
          :sort-desc="[true]"
          :items-per-page="10"
          :footer-props="{'items-per-page-options': [10, 25, 50, 100]}"
          :server-items-length="getNumberPendingDevices"
          :options.sync="pagination"
          :disable-sort="true"
          :search="search"
        >
          <template slot="no-data">
            There are no more pending devices
          </template>

          <template v-slot:item.hostname="{ item }">
            <router-link :to="{ name: 'detailsDevice', params: { id: item.uid } }">
              {{ item.name }}
            </router-link>
          </template>

          <template v-slot:item.info.pretty_name="{ item }">
            <DeviceIcon :icon-name="item.info.id" />
            {{ item.info.pretty_name }}
          </template>

          <template v-slot:item.actions="{ item }">
            <DeviceAccept
              :uid="item.uid"
              :state-list="true"
              @update="refresh"
            />
          </template>
        </v-data-table>
      </v-card-text>
    </v-card>
  </fragment>
</template>

<script>

import DeviceAdd from '@/components/device/DeviceAdd';
import DeviceIcon from '@/components/device//DeviceIcon';
import DeviceAccept from '@/components/device//DeviceAccept';

export default {
  name: 'DevicePendingList',

  components: {
    DeviceAdd,
    DeviceIcon,
    DeviceAccept,
  },

  data() {
    return {
      pagination: {},
      search: '',
      headers: [
        {
          text: 'Hostname',
          value: 'hostname',
          align: 'center',
        },
        {
          text: 'Operating System',
          value: 'info.pretty_name',
          align: 'center',
        },
        {
          text: 'Actions',
          value: 'actions',
          align: 'center',
          sortable: false,
        },
      ],
    };
  },

  computed: {
    getListPendingDevices() {
      return this.$store.getters['devices/list'];
    },

    getNumberPendingDevices() {
      return this.$store.getters['devices/getNumberDevices'];
    },
  },

  watch: {
    pagination: {
      handler() {
        this.getPendingDevices();
      },
      deep: true,
    },

    search() {
      this.getPendingDevices();
    },
  },

  methods: {
    getPendingDevices() {
      let filter = null;
      let encodedFilter = null;

      if (this.search) {
        filter = [{ type: 'property', params: { name: 'name', operator: 'like', value: this.search } }];
        encodedFilter = btoa(JSON.stringify(filter));
      }

      const data = {
        perPage: this.pagination.itemsPerPage,
        page: this.pagination.page,
        filter: encodedFilter,
        status: 'pending',
      };

      this.$store.dispatch('devices/fetch', data);
    },

    refresh() {
      this.getPendingDevices();
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
