<script>
  import { sourceQuery } from '../stores.js'
  import { debounce } from '../utils.js'
  import { parse, print } from 'graphql'

  const IS_LOADING_MESSAGE = 'Loading...'

  let code = ' '
  let variables = null
  let isLoading = false
  let error = null

  const debouncedUpdate = debounce(value => {
    if (value) {
      try {
        const normalizedString = value.replace(/(\r\n|\n|\r)/gm, "")
        const parsedString = JSON.parse(normalizedString)

        const isVariablesExists = !!Object.keys(parsedString.variables).length

        code = print(parse(parsedString.query))
        if (isVariablesExists) {
          variables = JSON.stringify(parsedString.variables, null, 2)
        } else {
          variables = null
        }
      } catch (e) {
        console.log(e)
        error = 'Can\'t parce input :('
      } finally {
        isLoading = false
      }
    }
  }, 500)

  sourceQuery.subscribe(value => {
    if (value) {
      error = null
      isLoading = true
    }
    debouncedUpdate(value)
  });
</script>

<style>
  pre {
    overflow-y: auto;
    white-space: pre-wrap;
    line-height: 1.5;
  }

  span {
    font-size: 20px;
  }

  .content, .message {
    height: 100%;
  }

  .content {
    display: flex;
    flex-direction: column;
  }

  .variables {
    position: relative;
    min-height: 200px;
    overflow-y: auto;
    border-top: 2px dashed var(--main-border-color);
  }

  .code, .variables {
    padding: 20px;
  }

  .message {
    display: flex;
    align-items: center;
    justify-content: center;
  }
</style>

{#if !isLoading && !error && code}
  <div class="content">
    <pre class="code">
      {@html code}
    </pre>
    {#if variables}
      <pre class="variables">
        {@html variables}
      </pre>
    {/if}
  </div>
{:else}
  <div class="message">
    <span>{error || IS_LOADING_MESSAGE}</span>
  </div>
{/if}



