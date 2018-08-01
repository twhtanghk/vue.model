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
Vue.use require('bootstrap-vue')
Vue.use require('vue.oauth2/src/plugin').default
eventBus = require('vue.oauth2/src/eventBus').default

export default
  components:
    model:
      extends: require('./model').default
      methods:
        list: (opts = {}) ->
          opts.method = 'GET'
          opts.url = @baseUrl
          res = await @fetch opts
          res.results?.map @format

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
        for i in value
          @collection.push i
</script>
