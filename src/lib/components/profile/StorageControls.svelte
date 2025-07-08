<!-- src/lib/components/profile/StorageControls.svelte -->
<script lang="ts">
  import { Button } from 'flowbite-svelte';
  import { DownloadOutline, UploadOutline, TrashBinOutline } from 'flowbite-svelte-icons';
  import { createEventDispatcher } from 'svelte';

  // Props
  export let profile_name: string = '';

  // Event dispatcher for parent communication
  const dispatch = createEventDispatcher<{
    export: void;
    import: { file: File };
    clearAll: void;
  }>();

  // File input reference
  let fileInput: HTMLInputElement | null = null;

  // Event handlers
  function handleExport() {
    dispatch('export');
  }

  function handleImportClick() {
    fileInput?.click();
  }

  function handleImportFile(event: Event) {
    const input = event.target as HTMLInputElement;
    const file = input.files && input.files[0];
    if (file) {
      dispatch('import', { file });
      // Clear the input
      input.value = '';
    }
  }

  function handleClearAll() {
    dispatch('clearAll');
  }
</script>

<!-- Storage Controls Section -->
<div class="mb-6 p-4 bg-gray-50 dark:bg-gray-700 rounded-lg border">
  <div class="flex flex-wrap items-center justify-between gap-4">
    <div>
      <h3 class="text-sm font-medium text-gray-900 dark:text-white">Profile Data</h3>
      <p class="text-xs text-gray-600 dark:text-gray-300">Auto-saved locally • Export for use in other apps</p>
    </div>
    <div class="flex flex-wrap gap-2">
      <Button color="light" size="sm" onclick={handleExport}>
        <DownloadOutline class="w-4 h-4 mr-2" />
        Export JSON
      </Button>
      
      <!-- Hidden file input -->
      <input 
        type="file" 
        accept=".json" 
        onchange={handleImportFile} 
        bind:this={fileInput}
        class="hidden" 
      />
      
      <Button color="light" size="sm" onclick={handleImportClick}>
        <UploadOutline class="w-4 h-4 mr-2" />
        Import JSON
      </Button>
      
      <Button color="red" size="sm" onclick={handleClearAll}>
        <TrashBinOutline class="w-4 h-4 mr-2" />
        Clear All
      </Button>
    </div>
  </div>
</div>