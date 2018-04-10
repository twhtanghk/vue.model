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
    mw: [
      @methodOverride
      @token
      @req
    ]
  methods:
    use: (middleware) ->
      @mw.push middleware
    form: (opts = {}) ->
      opts.headers ?= {}
      opts.headers['Content-Type'] = 'application/x-www-form-urlencoded'
      if opts.data?
        param = new URLSearchParams()
        for k, v of opts.data
          param.set k, v
        if opts.method == 'GET'
          opts.url += "?#{param}"
        else
          opts.body = param
      opts
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
      res = await @fetch opts
      if Array.isArray res
        res.map @format
      else
        @format res
    get: (opts = {}) ->
      opts.method = 'GET'
      res = await @fetch opts
      if Array.isArray res
        res.map @format
      else
        @format res
    put: (opts = {}) ->
      opts.method = 'PUT'
      res = await @fetch opts
      if Array.isArray res
        res.map @format
      else
        @format res
    del: (opts = {}) ->
      opts.method = 'DELETE'
      res = await @fetch opts
      if Array.isArray res
        res.map @format
      else
        @format res
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
</script>
