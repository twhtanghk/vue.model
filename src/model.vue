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
      opts.url ?= @baseUrl
      for i in @mw
        opts = await i opts
      opts
    post: (opts = {}) ->
      opts.method = 'POST'
      @format @fetch opts
    get: (opts = {}) ->
      opts.method = 'GET'
      @format @fetch opts
    put: (opts = {}) ->
      opts.method = 'PUT'
      @format @fetch opts
    del: (opts = {}) ->
      opts.method = 'DELETE'
      @fetch opts
    listAll: (opts = {}) -> 
      =>
        next: (skip = 0) =>
          opts.data ?= {}
          opts.data.skip = skip
          @list opts
            .then (page) ->
              done: page.length == 0
              value: page
    count: (opts = {}) ->
      opts.url ?= "#{@baseUrl}/count"
      {count} = @get opts
      count
    list: (opts = {}) ->
      @get opts
    create: (opts = {}) ->
      @post opts
    read: (id, opts = {}) ->
      opts.url = "#{@baseUrl}/#{id}"
      @get opts
    update: (id, opts = {}) ->
      opts.url = "#{@baseUrl}/#{id}"
      @put opts
    'delete': (id, opts = {}) ->
      opts.url = "#{@baseUrl}/#{id}"
      @del opts
  created: ->
    @mw.push @methodOverride
    @mw.push @token
    @mw.push @req
</script>
