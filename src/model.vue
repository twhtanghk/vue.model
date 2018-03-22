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
    'oauth2'
  ]
  methods:
    param: (data) ->
      ret = new URLSearchParams()
      for k, v of data
        ret.set k, v
      ret
    token: ->
      new Promise (resolve, reject) =>
        @eventBus
          .$emit 'oauth2.getToken'
          .$on 'oauth2.token', resolve
          .$on 'oauth2.error', reject
    req: (method, url, opts) ->
      valid = (res) =>
        if res.status != 200
          throw new Error res.statusText
        res
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
    list: (opts = {}) ->
      @token()
        .then (token) =>
          opts.headers ?= {}
          _.defaults opts.headers, Authorization: "Bearer #{token}"
          @req 'GET', @baseUrl, opts
    read: (id, opts = {}) ->
      @token()
        .then (token) =>
          opts.headers ?= {}
          _.defaults opts.headers, Authorization: "Bearer #{token}"
          @req 'GET', "#{@baseUrl}/#{id}", opts
    create: (opts) ->
      @token()
        .then (token) =>
          opts.headers ?= {}
          _.defaults opts.headers, Authorization: "Bearer #{token}"
          @req 'POST', @baseUrl, opts
    update: (id, opts) ->
      @token()
        .then (token) =>
          opts.headers ?= {}
          _.defaults opts.headers, Authorization: "Bearer #{token}"
          @req 'PUT', "#{@baseUrl}/#{id}", opts
    'delete': (opts) ->
      @token()
        .then (token) =>
          opts.headers ?= {}
          _.defaults opts.headers, Authorization: "Bearer #{token}"
          @req 'DELETE', "#{@baseUrl}/#{id}", opts
</script>
