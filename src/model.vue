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
    token: (opts) ->
      if opts.headers?.Authorization?
        return Promise.resolve opts
      new Promise (resolve, reject) =>
        @eventBus
          .$emit 'oauth2.getToken'
          .$on 'oauth2.token', (token) ->
            opts.headers ?= {}
            opts.headers.Authorization = "Bearer #{token}"
            resolve opts
          .$on 'oauth2.error', reject
    req: (method, url, opts) ->
      valid = (res) =>
        if res.status != 200
          throw new Error res.statusText
        res.json()
      opts.headers['X-HTTP-Method-Override'] = method
      opts.method = 'POST'
      opts.body = JSON.stringify opts.data
      fetch url, opts
        .then valid
    count: (opts = {}) ->
      @token opts
        .then (opts) =>
          @req 'GET', "#{@baseUrl}/count", opts
        .then (res) ->
          res.count
    listAll: (opts = {}) ->
      ret = []
      @token opts
        .then (opts) =>
          @count opts  
        .then (count) =>
          opts.data ?= {}
          opts.data.skip = ret.length
          cond = ->
            ret.length < count
          promiseWhile = require 'ya-promise-while'
          promiseWhile cond, =>
            @list opts
              .then (res) ->
                ret = ret.concat res
                opts.data.skip = ret.length
        .then ->
          ret
    list: (opts = {}) ->
      @token opts
        .then (opts) =>
          @req 'GET', @baseUrl, opts
    read: (id, opts = {}) ->
      @token opts
        .then (opts) =>
          @req 'GET', "#{@baseUrl}/#{id}", opts
    create: (opts = {}) ->
      @token opts
        .then (opts) =>
          @req 'POST', @baseUrl, opts
    update: (id, opts = {}) ->
      @token opts
        .then (opts) =>
          @req 'PUT', "#{@baseUrl}/#{id}", opts
    'delete': (opts = {}) ->
      @token opts
        .then (opts) =>
          @req 'DELETE', "#{@baseUrl}/#{id}", opts
</script>
