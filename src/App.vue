<template>
  <ul id="app">
    <li v-for='i in collection'>{{i}}</li>
  </ul>
</template>

<script lang='coffee'>
require './app.scss'

Vue = require('vue').default
Vue.use require('bootstrap-vue').default
Vue.use require('vue.oauth2/plugin').default
Vue.use require('./plugin.coffee').default

module.exports =
  data: ->
    collection: []
  
</script>
