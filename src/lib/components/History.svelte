<style>
p {
  white-space: pre-line;
}
</style>

<script lang="ts">
import { Diamond, Sprout, XCircle, Loader } from 'lucide-svelte'
import { Button } from '$components/ui/button'
import { Card, CardContent } from '$components/ui/card'
import { HistoryState, StreamState } from '$lib/store/history'
import { invoke } from '@tauri-apps/api/tauri'
import { toasts } from '$lib/store/toasts'

async function stopGenerating() {
  await invoke('stop_inference').catch((err) => {
    toasts.error(err)
  })
}
</script>

<Card
  class="relative w-[80%] mx-auto my-2 mt-14 overflow-auto no-scrollbar">
  <CardContent class="my-4 h-[80vh]">
    {#each $HistoryState as message, idx (idx)}
      {#if message.role == 'Me'}
        <div class=" flex w-[60%] items-center space-x-4 mb-3 float-right">
          <div class="flex-1 space-y-1 border rounded-md p-4">
            <p class="text-sm text-muted-foreground">{message.text}</p>
          </div>
          <Sprout size="{12}" />
        </div>
      {:else}
        <div class=" flex w-[60%] items-center space-x-4 mb-3">
          <Diamond size="{12}" />
          <div class="flex-1 space-y-1 border rounded-md p-4">
            <p class="text-sm text-muted-foreground">{message.text}</p>
          </div>
        </div>
      {/if}
    {/each}
    {#if $StreamState.isStreaming}
      <div class=" flex w-[60%] items-center space-x-4 mb-3">
        <Diamond size="{12}" />
        <div class="flex-1 space-y-1 border rounded-md p-4">
          <p class="text-sm text-muted-foreground">{$StreamState.tokens}</p>
        </div>
      </div>
    {/if}
  </CardContent>
  {#if $StreamState.isStreaming}
    <Button
      class="sticky w-[30%] mx-[33.33%] bottom-2 opacity-50 group animate-in animate-out"
      on:click="{stopGenerating}">
      <XCircle class="mr-2 h-4 w-4 pointer-events-none" /> Stop generating...
    </Button>
  {/if}
  {#if $StreamState.isFeedingPrompt}
    <Button
      class="sticky w-[30%] mx-[33.33%] bottom-2 opacity-50 group animate-in animate-out">
      <Loader class="mr-2 h-4 w-4 pointer-events-none" /> Feeding prompt...
    </Button>
  {/if}
</Card>
