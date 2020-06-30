<template>
  <v-menu
    offset-y
    :close-on-content-click="false"
    :value="shown"
    :disabled="getStatusNotifications"
  >
    <template v-slot:activator="{ on }">
      <v-badge
        :content="getNumberNotifications"
        :value="getNumberNotifications"
        overlap
        offset-x="20"
        color="success"
      >
        <v-chip
          v-on="on"
          @click="shown=true"
        >
          <v-icon>
            notifications
          </v-icon>
        </v-chip>
      </v-badge>
    </template>

    <v-card>
      <v-subheader>Pending Devices</v-subheader>

      <v-divider />

      <v-list-item
        v-for="(item, index) in getListNotifications"
        :key="index"
        router
        :to="item.path"
      >
        <v-list-item-content>
          <v-list-item-title>
            {{ item.name }}
          </v-list-item-title>
        </v-list-item-content>

        <v-list-item-action>
          <DeviceAccept
            :uid="item.uid"
            @update="refresh"
          />
        </v-list-item-action>
      </v-list-item>

      <v-divider />

      <v-btn
        to="/devices/pending"
        block
        link
        small
        @click="shown=false"
      >
        Show all Pending Devices
      </v-btn>
    </v-card>
  </v-menu>
</template>

<script>

import DeviceAccept from '@/components/device/DeviceAccept';

export default {
  name: 'NotificationIcon',

  components: {
    DeviceAccept,
  },

  data() {
    return {
      listNotifications: [],
      numberNotifications: 0,
      shown: false,
    };
  },

  computed: {
    getListNotifications() {
      return this.$store.getters['notifications/list'];
    },

    getNumberNotifications() {
      return this.$store.getters['notifications/getNumberNotifications'];
    },

    getStatusNotifications() {
      if (this.getNumberNotifications === 0) { return true; }
      return false;
    },
  },

  async created() {
    await this.getNotifications();
  },

  methods: {
    getNotifications() {
      this.$store.dispatch('notifications/fetch');
    },

    refresh() {
      this.getNotifications();
    },
  },
};

</script>
