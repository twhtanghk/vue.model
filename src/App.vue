<template>
  <div id="app">
    <authForm :eventBus='eventBus' :oauth2='oauth2' />
    <model ref='user' :eventBus='eventBus' baseUrl='https://app.ogcio.gov.hk/im/api/user' />
    <ul>
      <li v-for='item in collection' :key='item.id'>
        {{item.username}}
      </li>
    </ul>
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
  mounted: ->
    gen = await @$refs.user.listAll 
      data:
        sort:
          email: 1
    do =>
      {next} = gen()
      while true
        {done, value} = await next @collection.length
        break if done
        {count, results} = value
        for i in results
          @collection.push i
</script>
