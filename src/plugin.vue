<script lang='coffee'>
module.exports =
  install: (Vue, opts) ->
    Vue.component 'model', Vue.extend require('./model').default
</script>
