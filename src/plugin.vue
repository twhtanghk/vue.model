<script lang='coffee'>
export default
  install: (Vue, opts) ->
    Vue.component 'model', Vue.extend require('./model').default
</script>
