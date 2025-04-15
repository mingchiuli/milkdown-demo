<script setup lang="ts">
import { Milkdown, useEditor } from '@milkdown/vue'
import { Crepe } from '@milkdown/crepe'
import { Doc } from 'yjs'
import { WebsocketProvider } from 'y-websocket'
import { IndexeddbPersistence } from 'y-indexeddb'
import { collab, collabServiceCtx } from '@milkdown/plugin-collab'
import * as random from 'lib0/random'

const markdown = `# Milkdown Vue Crepe

> You're scared of a world where you're needed.

This is a demo for using Crepe with **Vue**.`

useEditor((root) => {
  const crepe = new Crepe({
    root,
  })

  const editor = crepe.editor
  editor.use(collab)
  crepe.create().then(() => {
    const doc = new Doc()
    // 创建 IndexedDB 持久化实例
    const indexeddbProvider = new IndexeddbPersistence('milkdown', doc)

    indexeddbProvider.whenSynced.then(() => {
      const wsProvider = new WebsocketProvider('ws://localhost:8089/rooms', 'milkdown', doc, {
        params: {
          token: localStorage.getItem('accessToken')!,
        },
      })

      const usercolors = [
        { color: '#30bced', light: '#30bced33' },
        { color: '#6eeb83', light: '#6eeb8333' },
        { color: '#ffbc42', light: '#ffbc4233' },
        { color: '#ecd444', light: '#ecd44433' },
        { color: '#ee6352', light: '#ee635233' },
        { color: '#9ac2c9', light: '#9ac2c933' },
        { color: '#8acb88', light: '#8acb8833' },
        { color: '#1be7ff', light: '#1be7ff33' },
      ]

      const userColor = usercolors[random.uint32() % usercolors.length]

      wsProvider.awareness.setLocalStateField('user', {
        name: 'Anonymous ' + Math.floor(Math.random() * 100),
        color: userColor.color,
        colorLight: userColor.light,
      })

      editor.action((ctx) => {
        const collabService = ctx.get(collabServiceCtx)

        collabService.bindDoc(doc).setAwareness(wsProvider.awareness)

        wsProvider.once('sync', async (isSynced: boolean) => {
          if (isSynced) {
            collabService
              .applyTemplate(markdown, (remoteNode, templateNode) => {
                return (
                  !remoteNode.textContent || remoteNode.textContent === templateNode.textContent
                )
              })
              .connect()
          }
        })
      })
    })
  })

  return crepe
})
</script>

<template>
  <Milkdown />
</template>
