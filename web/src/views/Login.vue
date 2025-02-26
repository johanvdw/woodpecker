<template>
  <div class="flex flex-col w-full h-full justify-center items-center">
    <div v-if="errorMessage" class="bg-red-400 text-white dark:text-gray-500 p-4 rounded-md text-lg">
      {{ errorMessage }}
    </div>

    <div
      class="
        flex flex-col
        w-full
        overflow-hidden
        md:m-8 md:rounded-md md:shadow md:border md:bg-white md:dark:bg-dark-gray-700
        dark:border-dark-200
        md:flex-row md:w-3xl md:h-sm
        justify-center
      "
    >
      <div class="flex md:bg-lime-500 md:dark:bg-lime-900 md:w-3/5 justify-center items-center">
        <img class="w-48 h-48" src="../assets/logo.svg?url" />
      </div>
      <div class="flex flex-col my-8 md:w-2/5 p-4 items-center justify-center">
        <h1 class="text-xl text-gray-600 dark:text-gray-500">Welcome to Woodpecker</h1>
        <Button class="mt-4" @click="doLogin">Login</Button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref } from 'vue';
import { useRoute, useRouter } from 'vue-router';

import Button from '~/components/atomic/Button.vue';
import useAuthentication from '~/compositions/useAuthentication';

const authErrorMessages = {
  oauth_error: 'Error while authenticating against OAuth provider',
  internal_error: 'Some internal error occured',
  access_denied: 'You are not allowed to login',
};

export default defineComponent({
  name: 'Login',

  components: {
    Button,
  },

  setup() {
    const route = useRoute();
    const router = useRouter();
    const authentication = useAuthentication();
    const errorMessage = ref<string>();

    function doLogin() {
      const url = typeof route.query.origin === 'string' ? route.query.origin : '';
      authentication.authenticate(url);
    }

    onMounted(async () => {
      if (authentication.isAuthenticated) {
        await router.replace({ name: 'home' });
        return;
      }

      if (route.query.code) {
        const code = route.query.code as keyof typeof authErrorMessages;
        errorMessage.value = authErrorMessages[code];
      }
    });

    return {
      doLogin,
      errorMessage,
    };
  },
});
</script>
