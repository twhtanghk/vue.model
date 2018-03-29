<template>
</template>

<script lang='coffee'>
Vue = require('vue').default
Vue.use require('vue.oauth2/src/plugin').default
eventBus = require('vue.oauth2/src/eventBus').default

module.exports =
  props: [
    'eventBus'
    'baseUrl'
  ]
  methods:
    getToken: ->
      new Promise (resolve, reject) =>
        @eventBus
          .$emit 'oauth2.getToken'
          .$on 'oauth2.token', (token) =>
            @token = token
            resolve @token
          .$on 'oauth2.error', reject
    opts: (opts) ->
      if opts.headers?.Authorization?
        return Promise.resolve opts
      @getToken()
        .then (token) ->
          opts.headers ?= {}
          opts.headers.Authorization = "Bearer #{token}"
          Promise.resolve opts
    req: (method, url, opts) ->
      opts.headers['X-HTTP-Method-Override'] = method
      opts.method = 'POST'
      opts.body = JSON.stringify opts.data
      res = await fetch url, opts
      if res.status != 200
        throw new Error res.statusText
      res.json()
    count: (opts = {}) ->
      {count} = @req 'GET', "#{@baseUrl}/count", await @opts opts
      count
    listAll: (opts = {}) -> 
      opts = await @opts opts
      =>
        next: (skip = 0) =>
          opts.data.skip = skip
          @list opts
            .then (page) ->
              done: page.length == 0
              value: page
    list: (opts = {}) ->
      @req 'GET', @baseUrl, await @opts opts
    read: (id, opts = {}) ->
      @req 'GET', "#{@baseUrl}/#{id}", await @opts opts
    create: (opts = {}) ->
      @req 'POST', @baseUrl, await @opts opts
    update: (id, opts = {}) ->
      @req 'PUT', "#{@baseUrl}/#{id}", await @opts opts
    'delete': (opts = {}) ->
      @req 'DELETE', "#{@baseUrl}/#{id}", await @opts opts
</script>
