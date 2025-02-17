:::warning 현재 이 문서는 번역 작업이 진행중입니다
:::

# Server-Side Rendering API

## renderToString()

- **Exported from `vue/server-renderer`**

- **타입**:

  ```ts
  function renderToString(
    input: App | VNode,
    context?: SSRContext
  ): Promise<string>
  ```

- **예제**:

  ```js
  import { createSSRApp } from 'vue'
  import { renderToString } from 'vue/server-renderer'

  const app = createSSRApp({
    data: () => ({ msg: 'hello' }),
    template: `<div>{{ msg }}</div>`
  })

  ;(async () => {
    const html = await renderToString(app)
    console.log(html)
  })()
  ```

  ### SSR Context

  You can pass an optional context object, which can be used to record additional data during the render, for example [accessing content of Teleports](/guide/scaling-up/ssr.html#teleports):

  ```js
  const ctx = {}
  const html = await renderToString(app, ctx)

  console.log(ctx.teleports) // { '#teleported': 'teleported content' }
  ```

  Most other SSR APIs on this page also optionally accept a context object. The context object can be accessed in component code via the [useSSRContext](#usessrcontext) helper.

- **참고**: [가이드 - Server-Side Rendering](/guide/scaling-up/ssr.html)

## renderToNodeStream()

Renders input as a [Node.js Readable stream](https://nodejs.org/api/stream.html#stream_class_stream_readable).

- **Exported from `vue/server-renderer`**

- **타입**:

  ```ts
  function renderToNodeStream(
    input: App | VNode,
    context?: SSRContext
  ): Readable
  ```

- **예제**:

  ```js
  // inside a Node.js http handler
  renderToNodeStream(app).pipe(res)
  ```

  :::tip Note
  This method is not supported in the ESM build of `vue/server-renderer`, which is decoupled from Node.js environments. Use [`pipeToNodeWritable`](#pipetonodewritable) instead.
  :::

## pipeToNodeWritable()

Render and pipe to an existing [Node.js Writable stream](https://nodejs.org/api/stream.html#stream_writable_streams) instance.

- **Exported from `vue/server-renderer`**

- **타입**:

  ```ts
  function pipeToNodeWritable(
    input: App | VNode,
    context: SSRContext = {},
    writable: Writable
  ): void
  ```

- **예제**:

  ```js
  // inside a Node.js http handler
  pipeToNodeWritable(app, {}, res)
  ```

## renderToWebStream()

Renders input as a [Web ReadableStream](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API).

- **Exported from `vue/server-renderer`**

- **타입**:

  ```ts
  function renderToWebStream(
    input: App | VNode,
    context?: SSRContext
  ): ReadableStream
  ```

- **예제**:

  ```js
  // inside an environment with ReadableStream support
  return new Response(renderToWebStream(app))
  ```

  :::tip Note
  In environments that do not expose `ReadableStream` constructor in the global scope, [`pipeToWebWritable()`](#pipetowebwritable) should be used instead.
  :::

## pipeToWebWritable()

Render and pipe to an existing [Web WritableStream](https://developer.mozilla.org/en-US/docs/Web/API/WritableStream) instance.

- **Exported from `vue/server-renderer`**

- **타입**:

  ```ts
  function pipeToWebWritable(
    input: App | VNode,
    context: SSRContext = {},
    writable: WritableStream
  ): void
  ```

- **예제**:

  This is typically used in combination with [`TransformStream`](https://developer.mozilla.org/en-US/docs/Web/API/TransformStream):

  ```js
  // TransformStream is available in environments such as CloudFlare workers.
  // in Node.js, TransformStream needs to be explicitly imported from 'stream/web'
  const { readable, writable } = new TransformStream()
  pipeToWebWritable(app, {}, writable)

  return new Response(readable)
  ```

## renderToSimpleStream()

Renders input in streaming mode using a simple readable interface.

- **Exported from `vue/server-renderer`**

- **타입**:

  ```ts
  function renderToSimpleStream(
    input: App | VNode,
    context: SSRContext,
    options: SimpleReadable
  ): SimpleReadable

  interface SimpleReadable {
    push(content: string | null): void
    destroy(err: any): void
  }
  ```

- **예제**:

  ```js
  let res = ''

  renderToSimpleStream(
    app,
    {},
    {
      push(chunk) {
        if (chunk === null) {
          // done
          console(`render complete: ${res}`)
        } else {
          res += chunk
        }
      },
      destroy(err) {
        // error encountered
      }
    }
  )
  ```

## useSSRContext()

A runtime API used to retrieve the context object passed to `renderToString()` or other server render APIs.

- **타입**:

  ```ts
  function useSSRContext<T = Record<string, any>>(): T | undefined
  ```

- **예제**:

  The retrieved context can be used to attach information that is needed for rendering the final HTML (e.g. head metadata).

  ```vue
  <script setup>
  import { useSSRContext } from 'vue'

  // make sure to only call it during SSR
  // https://vitejs.dev/guide/ssr.html#conditional-logic
  if (import.meta.env.SSR) {
    const ctx = useSSRContext()
    // ...attach properties to the context
  }
  </script>
  ```
