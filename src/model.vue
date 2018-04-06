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
  data: ->
    mw: []
  methods:
    use: (middleware) ->
      @mw.push middleware
    methodOverride: (opts = {}) ->
      opts.headers ?= {}
      opts.headers['X-HTTP-Method-Override'] = opts.method
      opts.method = 'POST'
      opts.body = JSON.stringify opts.data
      opts
    token: (opts = {}) ->
      token = await do => new Promise (resolve, reject) =>
        @eventBus
          .$once 'oauth2.token', resolve
          .$once 'oauth2.error', reject
          .$emit 'oauth2.getToken'
      opts.headers ?= {}
      opts.headers.Authorization = "Bearer #{token}"
      opts
    req: (opts = {}) ->
      res = await fetch opts.url, opts
      if res.status != 200
        throw new Error res.statusText
      res.json()
    format: (data) ->
      data
    fetch: (opts = {}) ->
      for i in @mw
        opts = await i opts
      opts
    count: (opts = {}) ->
      opts.url ?= "#{@baseUrl}/count"
      opts.method ?= 'GET'
      {count} = @fetch opts
      count
    listAll: (opts = {}) -> 
      =>
        next: (skip = 0) =>
          opts.data ?= {}
          opts.data.skip = skip
          @list opts
            .then (page) ->
              done: page.length == 0
              value: page
    list: (opts = {}) ->
      opts.method = 'GET'
      opts.url = @baseUrl
      @fetch opts
        .map @format
    read: (id, opts = {}) ->
      opts.method = 'GET'
      opts.url = "#{@baseUrl}/#{id}"
      @format @fetch opts
    create: (opts = {}) ->
      opts.method = 'POST'
      opts.url = @baseUrl
      @format @fetch opts
    update: (id, opts = {}) ->
      opts.method = 'PUT'
      opts.url = "#{@baseUrl}/#{id}"
      @format @fetch opts
    'delete': (id, opts = {}) ->
      opts.method = 'DELETE'
      opts.url = "#{@baseUrl}/#{id}"
      @fetch opts
  created: ->
    @mw.push @methodOverride
    @mw.push @token
    @mw.push @req
</script>
