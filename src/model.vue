<template>
</template>

<script lang='coffee'>
Vue = require('vue').default
Vue.use require('vue.oauth2/src/plugin').default

export default
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
      @json
      @methodOverride
      @req
      @res
      @error 
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
    # middleware to set opt headers with default content-type as application/json
    json: (opts = {}) =>
      opts.headers ?= {}
      opts.headers['Content-Type'] = 'application/json'
      opts
    # middleware to override req method and allow get with json body data
    methodOverride: (opts = {}) ->
      opts.headers ?= {}
      opts.headers['X-HTTP-Method-Override'] ?= opts.method
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
      await fetch opts.url, opts
    error: (res) ->
      if res.status in [200..299]
        return res.data
      else if res.status == 401
        throw new Error "Unauthorized access"
      else
        throw new Error "#{res.statusText} #{JSON.stringify res.data}"
    # middleware to format res
    res: (res) ->
      res.data = await res.json()
      if Array.isArray res.data
        res.data = res.data.map @format
      else
        res.data = @format res.data
      res
    # default data transformation for extended module to override
    format: (data) ->
      data
    fetch: (opts = {}) ->
      opts.url ?= @baseUrl
      try
        for i in @mw
          opts = await i opts
        opts
      catch err
        if err.message == 'Unauthorized access'
          @mw.unshift @token
          return @fetch opts
        else
          throw err
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
    iterPage: (opts = {}) ->
      skip = 0
      while true
        opts.data ?= {}
        opts.data.skip = skip
        page = await @list opts
        skip += page.length
        if page.length == 0
          break
        yield page
    iterAll: (opts = {}) -> 
      for await page from @iterPage opts
        for i in page
          yield i
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
      await @get opts
    create: (opts = {}) ->
      await @post opts
    read: (opts = {}) ->
      opts.url ?= "#{@baseUrl}/#{opts.data[@idAttribute]}"
      await @get opts
    update: (opts = {}) ->
      opts.url ?= "#{@baseUrl}/#{opts.data[@idAttribute]}"
      await @put opts
    'delete': (opts = {}) ->
      opts.url ?= "#{@baseUrl}/#{opts.data[@idAttribute]}"
      await @del opts
</script>
