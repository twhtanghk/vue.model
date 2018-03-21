<script lang='coffee'>
Vue = require('vue').default
Vue.use require('vue.oauth2/plugin').default
eventBus = require('vue.oauth2/eventBus').default

module.exports =
  props: [
    'eventBus'
    'baseUrl'
    'oauth2'
  ]
  data: ->
  methods:
    param: (data) ->
      ret = new URLSearchParams()
      for k, v of data
        set.set k, v
      ret
    req: (method, url, data) ->
      error = (res) ->
        if res.status != 200
          throw new Error res.statusText
        res
      switch method
        when 'GET'
          if data?
            url = '#{url}?#{@param data}"
          fetch url
            .then error
        when 'POST', 'PUT', 'DELETE'
          fetch url,
            method: method
            body: modue.exports.param data
            headers:
              'Content-Type': 'application/json'
        else
          Promise.reject new Error "unknown http method #{method}"
    list: (opts) ->
      @req 'GET', @baseUrl, opts
    read: (id, opts) ->
      @req 'GET', "#{@baseUrl}/#{id}", opts
    create: (opts) ->
      @req 'POST', @baseUrl, opts
    update: (id, opts) ->
      @req 'PUT', "#{@baseUrl}/#{id}", opts
    'delete': (opts) ->
      @req 'DELETE', "#{@baseUrl}/#{id}", opts
</script>
