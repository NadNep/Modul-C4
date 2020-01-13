<template>
  <div>
    <b-alert v-model="showDismissibleAlert" variant="success" dismissible>
      {{ message }}
    </b-alert>
  </div>
</template>

<script>
export default {
  props: ['message', 'SendedAlertStatus'],
  data() {
    return {
      showDismissibleAlert: this.SendedAlertStatus
    }
  },
};
</script>
