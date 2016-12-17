<template>
  <div id="app">
    <Message>
    </Message>
  </div>
</template>

<script>
import Message from './components/Message'

export default {
  components: {
    Message
  }
}
</script>

<style>
    html,body {
      height: 100%;
    }
    body {
      background-color: rgba(153, 135, 140, 0.22);
    }
</style>
