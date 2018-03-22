<template>
</template>

<script lang='coffee'>
_ = require 'lodash'
Vue = require('vue').default
Vue.use require('vue.oauth2/src/plugin').default
eventBus = require('vue.oauth2/src/eventBus').default

module.exports =
  props: [
    'eventBus'
    'baseUrl'
  ]
  methods:
    param: (data) ->
      ret = new URLSearchParams()
      for k, v of data
        ret.set k, v
      ret
    token: (opts) ->
      if opts.headers?.Authorization?
        return Promise.resolve opts
      new Promise (resolve, reject) =>
        @eventBus
          .$emit 'oauth2.getToken'
          .$on 'oauth2.token', (token) ->
            opts.headers ?= {}
            _.defaults opts.headers, Authorization: "Bearer #{token}"
            resolve opts
          .$on 'oauth2.error', reject
    req: (method, url, opts) ->
      valid = (res) =>
        if res.status != 200
          throw new Error res.statusText
        res.json()
      _.defaults opts.headers,
        'Content-Type': 'application/json'
      switch method
        when 'GET'
          if opts.data?
            url = "#{url}?#{@param opts.data}"
          fetch url, headers: opts.headers
            .then valid
        when 'POST', 'PUT', 'DELETE'
          opts =
            method: method
            headers: opts.headers
            body: JSON.stringify opts.data
          fetch url, opts
            .then valid
        else
          Promise.reject new Error "unknown http method #{method}"
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
          skip = 0
          cond = ->
            ret.length < count
          promiseWhile = require 'ya-promise-while'
          promiseWhile cond, =>
            @list _.extend(opts, skip: skip)
              .then (res) ->
                ret = ret.concat res
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
