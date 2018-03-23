<template>
  <div id="app">
    <authForm :eventBus='eventBus' :oauth2='oauth2' />
    <model ref='portfolio' :eventBus='eventBus' baseUrl='http://172.23.0.3:1337/api/portfolio' />
    <b-table striped howver :items='collection' :fields='fields' />
  </div>
</template>

<script lang='coffee'>
Vue = require('vue').default
Vue.use require('bootstrap-vue').default
Vue.use require('vue.oauth2/src/plugin').default
Vue.use require('./plugin').default
eventBus = require('vue.oauth2/src/eventBus').default

module.exports =
  data: ->
    oauth2:
      url: 'https://app.ogcio.gov.hk/auth/oauth2/authorize/'
      client: 'testing'
      scope: 'User'
      response_type: 'token'
    eventBus: eventBus
    collection: []
    fields: [
      'symbol'
      'name'
      'type'
      'date'
      'quantity'
      'price'
      'tags'
    ]
  mounted: ->
    @$refs.portfolio.listAll data: sort: createdBy: 1
      .then (res) =>
        @collection = res
      .catch console.error
</script>
