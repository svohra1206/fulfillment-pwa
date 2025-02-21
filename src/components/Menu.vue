<template>
  <ion-menu side="start" content-id="main-content" type="overlay" :disabled="!isUserAuthenticated">
    <ion-header>
      <ion-toolbar>
        <ion-title>{{ currentFacility.facilityName }}</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <ion-list>
        <ion-menu-toggle auto-hide="false" v-for="(page, index) in getValidMenuItems(appPages)" :key="index">
          <ion-item
            button
            router-direction="root"
            :router-link="page.url"
            class="hydrated"
            :class="{ selected: selectedIndex === index }">
            <ion-icon slot="start" :ios="page.iosIcon" :md="page.mdIcon" />
            <ion-label>{{ translate(page.title) }}</ion-label>
          </ion-item>
        </ion-menu-toggle>
      </ion-list>
    </ion-content>
  </ion-menu>
</template>

<script lang="ts">
import {
  IonContent,
  IonHeader,
  IonIcon,
  IonItem,
  IonLabel,
  IonList,
  IonMenu,
  IonMenuToggle,
  IonTitle,
  IonToolbar
} from "@ionic/vue";
import { computed, defineComponent } from "vue";
import { mapGetters } from "vuex";
import { mailUnreadOutline, mailOpenOutline, checkmarkDoneOutline, settingsOutline, swapVerticalOutline } from "ionicons/icons";
import { useStore } from "@/store";
import { useRouter } from "vue-router";
import { hasPermission } from "@/authorization";
import { translate } from '@hotwax/dxp-components';

export default defineComponent({
  name: "Menu",
  components: {
    IonContent,
    IonHeader,
    IonIcon,
    IonItem,
    IonLabel,
    IonList,
    IonMenu,
    IonMenuToggle,
    IonTitle,
    IonToolbar
  },
  computed: {
    ...mapGetters({
      isUserAuthenticated: 'user/isUserAuthenticated',
      currentFacility: 'user/getCurrentFacility',
    })
  },
  methods: {
    getValidMenuItems(appPages: any) {
      return appPages.filter((appPage: any) => (!appPage.meta || !appPage.meta.permissionId) || hasPermission(appPage.meta.permissionId));
    }
  },
  setup() {
    const store = useStore();
    const router = useRouter();

    const appPages = [
      {
        title: "Open",
        url: "/open-orders",
        iosIcon: mailUnreadOutline,
        mdIcon: mailUnreadOutline,
        meta: {
          permissionId: "APP_OPEN_ORDERS_VIEW"
        }
      },
      {
        title: "In Progress",
        url: "/in-progress",
        iosIcon: mailOpenOutline,
        mdIcon: mailOpenOutline,
        meta: {
          permissionId: "APP_IN_PROGRESS_ORDERS_VIEW"
        }
      },
      {
        title: "Completed",
        url: "/completed",
        iosIcon: checkmarkDoneOutline,
        mdIcon: checkmarkDoneOutline,
        meta: {
          permissionId: "APP_COMPLETED_ORDERS_VIEW"
        }
      },
      {
        title: "EXIM",
        url: "/exim",
        iosIcon: swapVerticalOutline,
        mdIcon: swapVerticalOutline,
        childRoutes: ["/download-packed-orders", "/upload-import-orders", "/saved-mappings"], // defined child routes as to enable the correct menu when we are on a route that is not listed in the menu
        meta: {
          permissionId: "APP_EXIM_VIEW"
        }
      },
      {
        title: "Settings",
        url: "/settings",
        iosIcon: settingsOutline,
        mdIcon: settingsOutline,
      }
    ];

    const selectedIndex = computed(() => {
      const path = router.currentRoute.value.path
      return appPages.findIndex((screen) => screen.url === path || screen.childRoutes?.includes(path))
    })

    return {
      appPages,
      checkmarkDoneOutline,
      hasPermission,
      mailUnreadOutline,
      mailOpenOutline,
      selectedIndex,
      settingsOutline,
      store,
      translate
    };
  }
});
</script>

<style scoped>
ion-menu.md ion-item.selected ion-icon {
  color: var(--ion-color-secondary);
}

ion-menu.ios ion-item.selected ion-icon {
  color: var(--ion-color-secondary);
}

ion-item.selected {
  --color: var(--ion-color-secondary);
}
</style>