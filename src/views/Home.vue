<template>
  <v-main>
    <v-parallax
      height="320"
      :src="require('@/assets/' + system.status + '.jpg')"
    />

    <v-container class="mb-16">
      <v-alert
        prominent
        dense
        dark
        :color="status[system.status].color"
        :icon="status[system.status].icon"
      >
        <b v-if="system.status === 'None'">Some checks haven't completed yet</b>
        <b v-else-if="system.degraded">Some Systems Degraded</b>
        <b v-else-if="system.incident">Some Systems Incident</b>
        <b v-else>All Systems Operational</b>
      </v-alert>

      <v-card-title>
        Current status
        <v-spacer />
        <span class="caption text-right">
          {{ new Date() }}
        </span>
      </v-card-title>

      <v-row>
        <v-col cols="12" sm="6" v-for="service in services" :key="service.name">
          <v-card elevation="16">
            <v-list-item two-line>
              <v-list-item-avatar>
                <v-avatar :color="service.color" size="40">
                  <v-icon dark>{{ service.icon }}</v-icon>
                </v-avatar>
              </v-list-item-avatar>

              <v-list-item-content>
                <v-list-item-title>{{ service.title }}</v-list-item-title>

                <v-list-item-subtitle>
                  {{ service.status }}
                </v-list-item-subtitle>
              </v-list-item-content>

              <v-list-item-action>
                <v-tooltip bottom>
                  <template v-slot:activator="{ on, attrs }">
                    <v-icon
                      :color="status[service.status].color"
                      v-bind="attrs"
                      v-on="on"
                    >
                      {{ status[service.status].icon }}
                    </v-icon>
                  </template>
                  <span>
                    {{ service.desc ? service.desc : service.status }}
                  </span>
                </v-tooltip>
              </v-list-item-action>
            </v-list-item>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-main>
</template>

<script>
import {
  defineComponent,
  onBeforeMount,
  reactive,
  toRefs,
} from "@vue/composition-api"
import axios from "axios"

export default defineComponent({
  name: "Home",

  setup() {
    const state = reactive({
      system: {
        status: "None",
        degraded: false,
        incident: false,
      },
      status: {
        Normal: { icon: "mdi-check-circle", color: "green" },
        Degraded: { icon: "mdi-alert-circle", color: "orange" },
        Incident: { icon: "mdi-alert-circle", color: "red" },
        None: { icon: "mdi-checkbox-blank-circle-outline", color: "grey" },
      },
      services: [
        {
          color: "orange",
          icon: "mdi-cellphone-text",
          title: "Production",
          status: "None",
          desc: "",
          url: "https://api.hakhak.io/.well-known/apollo/server-health",
        },
        {
          color: "deep-purple accent-2",
          icon: "mdi-application",
          title: "Production Admin",
          status: "None",
          desc: "",
          url: "https://admin.hakhak.io/.well-known/apollo/server-health",
        },
        {
          color: "orange",
          icon: "mdi-cellphone-text",
          title: "Staging",
          status: "None",
          desc: "",
          url: "https://api-staging.hakhak.io/.well-known/apollo/server-health",
        },
        {
          color: "deep-purple accent-2",
          icon: "mdi-application",
          title: "Staging Admin",
          status: "None",
          desc: "",
          url: "https://admin-staging.hakhak.io/.well-known/apollo/server-health",
        },
        {
          color: "orange",
          icon: "mdi-cellphone-text",
          title: "Development",
          status: "None",
          desc: "",
          url: "https://api-dev.hakhak.io/.well-known/apollo/server-health",
        },
        {
          color: "deep-purple accent-2",
          icon: "mdi-application",
          title: "Development Admin",
          status: "None",
          desc: "",
          url: "https://admin-dev.hakhak.io/.well-known/apollo/server-health",
        },
      ],
    })

    const getStatus = async (e) => {
      await axios
        .get(e.url)
        .then(() => {
          e.status = "Normal"
        })
        .catch((err) => {
          state.system.degraded = true
          e.status = "Degraded"
          e.desc = err
        })
    }

    onBeforeMount(async () => {
      await Promise.all(state.services.map((e) => getStatus(e)))

      state.system.status = state.system.incident
        ? "Incident"
        : state.system.degraded
        ? "Degraded"
        : "Normal"

      document
        .getElementById("favicon")
        .setAttribute("href", state.system.status + ".svg")
    })

    return {
      ...toRefs(state),
    }
  },
})
</script>
