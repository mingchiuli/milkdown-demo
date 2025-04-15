<script setup lang="ts">
import { Milkdown, useEditor } from '@milkdown/vue'
import { Crepe } from '@milkdown/crepe'
import { Doc } from 'yjs'
import { WebsocketProvider } from 'y-websocket'
import { IndexeddbPersistence } from 'y-indexeddb'
import { collab, collabServiceCtx } from '@milkdown/plugin-collab'
import * as random from 'lib0/random'
import { onUnmounted, ref } from 'vue'
import type { Editor } from '@milkdown/kit/core'

const markdown = ref(`# Milkdown Vue Crepe

> You're scared of a world where you're needed.

This is a demo for using Crepe with **Vue**.`)
let indexeddbProvider: IndexeddbPersistence | undefined
let websocketProvider: WebsocketProvider | undefined
let editor: Editor | undefined

useEditor((root) => {
  const crepe = new Crepe({
    root,
    defaultValue: markdown.value // 设置初始值
  })

  editor = crepe.editor
  editor.use(collab)

  // 添加事件监听
  crepe.on((listener) => {
    listener.markdownUpdated((ctx, content) => {
      // 更新 markdown 的值
      markdown.value = content
    })
  })


  crepe.create().then(() => {
    const doc = new Doc()
    indexeddbProvider = new IndexeddbPersistence("123", doc)
    indexeddbProvider!.whenSynced.then(() => {
      websocketProvider = new WebsocketProvider(
        `ws://localhost:8089/rooms`,
        "123",
        doc,
        {
          params: {
            token: localStorage.getItem('accessToken')!
          }
        }
      )

      const usercolors = [
        { color: '#30bced', light: '#30bced33' },
        { color: '#6eeb83', light: '#6eeb8333' },
        { color: '#ffbc42', light: '#ffbc4233' },
        { color: '#ecd444', light: '#ecd44433' },
        { color: '#ee6352', light: '#ee635233' },
        { color: '#9ac2c9', light: '#9ac2c933' },
        { color: '#8acb88', light: '#8acb8833' },
        { color: '#1be7ff', light: '#1be7ff33' }
      ]

      const userColor = usercolors[random.uint32() % usercolors.length]

      websocketProvider.awareness.setLocalStateField('user', {
        name: "user.nickname",
        color: userColor.color,
        colorLight: userColor.light
      })

      editor!.action((ctx) => {
        const collabService = ctx.get(collabServiceCtx)
        collabService.bindDoc(doc).setAwareness(websocketProvider!.awareness)

        websocketProvider!.once('sync', async (isSynced: boolean) => {
          if (isSynced) {
            collabService
              .applyTemplate(markdown.value, (remoteNode, templateNode) => {
                console.log('remoteNode', remoteNode.textContent)
                console.log('localNode', templateNode.textContent)
                return !remoteNode.textContent
              })
              .connect()
            markdown.value = crepe.getMarkdown()

          }
        })
      })
    })
  })
  return crepe
})

onUnmounted(async () => {
  console.log("onUnmounted")
  if (websocketProvider) {
    websocketProvider.disconnect()
  }
  if (indexeddbProvider) {
    indexeddbProvider.destroy()
  }
})
</script>

<template>
  <Milkdown id="milk" />
  <div>
    {{ `data: ${markdown}` }}
  </div>
</template>