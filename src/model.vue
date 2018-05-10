<template>
</template>

<script lang='coffee'>
Vue = require('vue').default
Vue.use require('vue.oauth2/src/plugin').default

module.exports =
  props:
    baseUrl:
      type: String
    eventBus:
      type: Object
      default: require('vue.oauth2/src/eventBus').default
    idAttribute:
      type: String
      default: 'id'
  data: ->
    # default list of middleware to modify req opts
    # until req is processed by remote end
    mw: [
      @methodOverride
      @token
      @req
      @res
    ]
  methods:
    # middleware to set opts headers with content-type as x-www-form-urlencoded
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
    # middleware to override req method and allow get with json body data
    methodOverride: (opts = {}) ->
      opts.headers ?= {}
      opts.headers['X-HTTP-Method-Override'] = opts.method
      opts.method = 'POST'
      if not ('body' of opts and opts.body instanceof FormData)
        opts.body = JSON.stringify opts.data
      opts
    # middleware to acquire token from oauth2 module 
    # and embed token into req header for oauth2 authorization
    token: (opts = {}) ->
      token = await do => new Promise (resolve, reject) =>
        @eventBus
          .$once 'oauth2.token', resolve
          .$once 'oauth2.error', reject
          .$emit 'oauth2.getToken'
      opts.headers ?= {}
      opts.headers.Authorization = "Bearer #{token}"
      opts
    # middleware to send http req to remote end
    # and verify result status code
    # and return json body data
    req: (opts = {}) ->
      res = await fetch opts.url, opts
      if res.status not in [200..299]
        throw new Error res.statusText
      await res.json()
    # middleware to format res
    res: (res) ->
      if Array.isArray res
        res.map @format
      else
        @format res
    # default data transformation for extended module to override
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
    get: (opts = {}) ->
      opts.method = 'GET'
      res = await @fetch opts
    patch: (opts = {}) ->
      opts.method = 'PATCH'
      res = await @fetch opts
    put: (opts = {}) ->
      opts.method = 'PUT'
      res = await @fetch opts
    del: (opts = {}) ->
      opts.method = 'DELETE'
      res = await @fetch opts
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
    upload: (opts = {}) ->
      opts.method ?= 'POST'
      opts.url ?= "#{@baseUrl}/upload"
      formData = new FormData()
      for file in opts.files
        formData.append 'files', file
      opts.body = formData
      @fetch opts
    list: (opts = {}) ->
      @get opts
    create: (opts = {}) ->
      @post opts
    read: (opts = {}) ->
      opts.url ?= "#{@baseUrl}/#{opts.data[@idAttribute]}"
      @get opts
    update: (opts = {}) ->
      opts.url ?= "#{@baseUrl}/#{opts.data[@idAttribute]}"
      @put opts
    'delete': (opts = {}) ->
      opts.url ?= "#{@baseUrl}/#{opts.data[@idAttribute]}"
      @del opts
</script>
