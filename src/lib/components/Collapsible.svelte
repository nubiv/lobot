<script lang="ts">
import {
  ChevronDown,
  Play,
  StopCircle,
  ArrowDownToLine,
  Pause
} from 'lucide-svelte'
import { Button } from '$components/ui/button'
import {
  Collapsible,
  CollapsibleContent,
  CollapsibleTrigger
} from '$components/ui/collapsible'
import Progress from './ui/progress/Progress.svelte'
import { invoke } from '@tauri-apps/api/tauri'
import { DownloadState } from '$lib/store/download'
import { LLMState, type TModel } from '$lib/store/llm'
import DeleteDialog from './DeleteDialog.svelte'
import { toasts } from '$lib/store/toasts'

let isOpen = false
export let list: Record<string, TModel>
export let title: string

async function download(e: MouseEvent) {
  if ($DownloadState.currentDownload) {
    toasts.error('Currently only one download is supported.')
    return
  }

  const modelName = (e.target as HTMLButtonElement).id
  await invoke('download_model', { modelName }).catch((err) => {
    toasts.error(err)
  })

  const modelInfo = list[modelName]
  const { size, totalSize } = modelInfo
  const progress = ((size / totalSize) * 100.0).toFixed(2)

  DownloadState.startDownload(modelName, progress, totalSize)
}

async function stopDownload() {
  await invoke('stop_download').catch((err) => {
    toasts.error(err)
  })

  DownloadState.stopDownload()

  await invoke('update_llm_models').catch((err) => {
    toasts.error(err)
  })
}

async function loadModel(e: MouseEvent) {
  if ($LLMState.runnningModel) {
    toasts.error('Only one model can be run.')
    return
  }

  const modelName = (e.target as HTMLButtonElement).id
  await invoke('load_model', { modelName }).catch((err) => {
    toasts.error(err)
  })

  LLMState.updateRunningModel(modelName)
}

async function unloadModel() {
  await invoke('unload_model').catch((err) => {
    toasts.error(err)
  })

  LLMState.stopRunningModel()
}

// async function deleteModel(e: MouseEvent) {
//   const modelName = (e.target as HTMLButtonElement).id

//   if ($DownloadState.currentDownload === modelName) {
//     toasts.error('Stop downloading first...')
//     return
//   }

//   if ($LLMState.runnningModel === modelName) {
//     toasts.error('Stop running it first...')
//     return
//   }

//   await invoke('delete_model', { modelName })

//   await invoke('update_llm_models')
// }
</script>

<Collapsible open="{isOpen}" class="w-auto space-y-2 pt-5">
  <div class="flex items-center justify-between space-x-4 px-4">
    <h4 class="text-sm font-semibold">{title}</h4>
    <CollapsibleTrigger>
      <Button variant="ghost" size="sm" class="w-9 p-0">
        <ChevronDown class="h-4 w-4" />
        <span class="sr-only">Toggle</span>
      </Button>
    </CollapsibleTrigger>
  </div>
  <CollapsibleContent class="space-y-2 mx-5 pt-3">
    {#if Object.keys(list).length === 0}
      <div class="rounded-md overflow-auto px-4 py-3 font-mono text-sm">
        None
      </div>
    {:else}
      {#each Object.entries(list) as [modelName, modelInfo]}
        <div
          class="grid grid-cols-3 rounded-md overflow-auto py-1 font-mono text-sm">
          <div class="col-span-2">
            {modelName}
          </div>
          <div class="col-span-1">
            {#if title == 'Local Models'}
              {#if $LLMState.runnningModel === modelName}
                <Button
                  variant="ghost"
                  size="sm"
                  class="px-1 group"
                  on:click="{unloadModel}">
                  <StopCircle class="h-4 w-4 pointer-events-none" />
                  <span class="sr-only pointer-events-none">Stop</span>
                </Button>
              {:else}
                <Button
                  variant="ghost"
                  size="sm"
                  class="px-1 group"
                  id="{modelName}"
                  on:click="{(e) => loadModel(e)}">
                  <Play class="h-4 w-4 pointer-events-none" />
                  <span class="sr-only pointer-events-none">Start</span>
                </Button>
              {/if}
              <DeleteDialog modelName="{modelName}" />
              <!-- <Button
                variant="ghost"
                size="sm"
                class="px-1 group"
                id="{modelName}"
                on:click="{(e) => deleteModel(e)}">
                <Trash2 class="h-4 w-4 pointer-events-none" />
                <span class="sr-only pointer-events-none">Delete</span>
              </Button> -->
            {:else}
              {#if $DownloadState.currentDownload === modelName}
                <Button
                  variant="ghost"
                  size="sm"
                  class="px-1 group"
                  on:click="{stopDownload}">
                  <Pause class="h-4 w-4 pointer-events-none" />
                  <span class="sr-only pointer-events-none">Pause</span>
                </Button>
              {:else}
                <Button
                  variant="ghost"
                  size="sm"
                  class="px-1 group"
                  id="{modelName}"
                  on:click="{(e) => download(e)}">
                  <ArrowDownToLine class="h-4 w-4 pointer-events-none" />
                  <span class="sr-only pointer-events-none">Download</span>
                </Button>
              {/if}
              <DeleteDialog modelName="{modelName}" />
              <!-- <Button
                variant="ghost"
                size="sm"
                class="px-1 group"
                id="{modelName}"
                on:click="{(e) => deleteModel(e)}">
                <Trash2 class="h-4 w-4 pointer-events-none" />
                <span class="sr-only pointer-events-none">Delete</span>
              </Button> -->
            {/if}
          </div>
        </div>
        {#if title != 'Local Models'}
          <div class="grid grid-cols-10 gap-1">
            {#if $DownloadState.currentDownload == modelName}
              <Progress
                class=" col-span-9"
                value="{Number($DownloadState.progress)}" />
              <span class="col-span-1">{$DownloadState.progress}%</span>
            {:else}
              <!-- <Progress
                class=" col-span-9"
                value="{(modelInfo.size / modelInfo.totalSize) * 100.0}" />
              <span class="col-span-1"
                >{((modelInfo.size / modelInfo.totalSize) * 100.0).toFixed(
                  2
                )}%</span> -->
            {/if}
          </div>
        {/if}
      {/each}
    {/if}
  </CollapsibleContent>
</Collapsible>
