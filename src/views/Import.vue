<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Import Wallet</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <ion-item>
        <ion-label>Import Wallet</ion-label>
      </ion-item>
      <ion-item>
        <ion-text>{{ publicKey }}</ion-text>
      </ion-item>
      <ion-loading
        :is-open="loading"
        cssClass="my-custom-class"
        message="Please wait..."
        :duration="4000"
        :key="`k${loading}`"
        @didDismiss="loading = false"
      />
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonItem,
  IonLabel,
  IonText,
  IonLoading,
  onIonViewWillEnter,
} from "@ionic/vue";
import {saveAccount, saveSelectedAccount} from "@/utils/platform";
import { walletPing } from "@/extension/userRequest";
import { useRoute } from "vue-router";
import { blockLockout } from "@/utils/platform";
import {ethers} from "ethers";

export default defineComponent({
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonItem,
    IonLabel,
    IonText,
    IonLoading,
  },
  setup: () => {
    const route = useRoute();
    const loading = ref(false);
    let pk = route?.params?.param as string;
    const alertOpen = ref(false);
    const alertMsg = ref("");
    const timerReject = ref(140);
    let interval: any;
    let publicKey = "Invalid private key";
      if (pk.length === 64) {
          pk = `0x${pk.trim()}`;
      }
      if (pk.length !== 66) {
          alertMsg.value = "Provided private key is invalid.";
          alertOpen.value = true;
      } else {
          const wallet = new ethers.Wallet(pk);
          saveAccount({
              address: wallet.address,
              name: wallet.address,
              pk: pk,
              encPk: "",
          });
          saveSelectedAccount({
              address: wallet.address,
              name: wallet.address,
              pk: pk,
              encPk: "",
          });
          publicKey = wallet.address
      }
    onIonViewWillEnter(async () => {
      blockLockout();
      interval = setInterval(async () => {
        timerReject.value -= 1;
        walletPing();
      }, 1000) as any;
        setTimeout(() => {
            void(0);
        }, 200);
    });


    return {
        publicKey,
      alertOpen,
      alertMsg,
      loading,
      timerReject,
    };
  },
});
</script>
