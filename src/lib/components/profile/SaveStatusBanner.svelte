<!-- src/lib/components/profile/SaveStatusBanner.svelte -->
<script lang="ts">
  import { CheckCircleOutline, ExclamationCircleOutline } from 'flowbite-svelte-icons';
  import { createEventDispatcher } from 'svelte';

  // Props
  export let saveStatus: null | 'saving' | 'success' | 'error' = null;
  export let validationErrors: string[] = [];
  export let lastSaveTime: Date | null = null;
  export let lastSavedProfileId: string | null = null;
  export let saveLocation: 'local' | 'database' | null = null;
  export let detailedSaveMessage: string = '';

  // Event dispatcher
  const dispatch = createEventDispatcher<{
    dismiss: void;
  }>();

  function handleDismiss() {
    dispatch('dismiss');
  }
</script>

<!-- Save Status Banner -->
{#if saveStatus}
  <div class="mb-4 p-4 rounded-lg border transition-all duration-300 {
    saveStatus === 'saving' ? 'bg-blue-50 border-blue-200 dark:bg-blue-900/20 dark:border-blue-800' :
    saveStatus === 'success' ? 'bg-green-50 border-green-200 dark:bg-green-900/20 dark:border-green-800' :
    'bg-red-50 border-red-200 dark:bg-red-900/20 dark:border-red-800'
  }">
    
    <div class="flex items-start gap-3">
      <!-- Status Icon -->
      <div class="flex-shrink-0 mt-0.5">
        {#if saveStatus === 'saving'}
          <svg class="animate-spin h-5 w-5 text-blue-600 dark:text-blue-400" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
          </svg>
        {:else if saveStatus === 'success'}
          <CheckCircleOutline class="w-5 h-5 text-green-600 dark:text-green-400" />
        {:else if saveStatus === 'error'}
          <ExclamationCircleOutline class="w-5 h-5 text-red-600 dark:text-red-400" />
        {/if}
      </div>

      <!-- Status Content -->
      <div class="flex-1 min-w-0">
        <div class="flex items-center gap-2 mb-1">
          <h4 class="text-sm font-medium {
            saveStatus === 'saving' ? 'text-blue-800 dark:text-blue-200' :
            saveStatus === 'success' ? 'text-green-800 dark:text-green-200' :
            'text-red-800 dark:text-red-200'
          }">
            {#if saveStatus === 'saving'}
              Saving Profile...
            {:else if saveStatus === 'success'}
              {saveLocation === 'database' ? '✅ Database Save Successful!' : '✅ Local Save Successful!'}
            {:else if saveStatus === 'error'}
              {saveLocation === 'database' ? '❌ Database Save Failed' : '❌ Local Save Failed'}
            {/if}
          </h4>
          
          <!-- Save Location Badge -->
          {#if saveLocation}
            <span class="inline-flex items-center px-2 py-1 rounded-full text-xs font-medium {
              saveLocation === 'database' ? 'bg-purple-100 text-purple-800 dark:bg-purple-900/30 dark:text-purple-300' :
              'bg-gray-100 text-gray-800 dark:bg-gray-900/30 dark:text-gray-300'
            }">
              {saveLocation === 'database' ? '🗄️ Database' : '💾 Local Storage'}
            </span>
          {/if}
        </div>

        <!-- Detailed Message -->
        {#if detailedSaveMessage}
          <p class="text-sm {
            saveStatus === 'saving' ? 'text-blue-700 dark:text-blue-300' :
            saveStatus === 'success' ? 'text-green-700 dark:text-green-300' :
            'text-red-700 dark:text-red-300'
          }">
            {detailedSaveMessage}
          </p>
        {/if}

        <!-- Timestamp -->
        {#if lastSaveTime && saveStatus === 'success'}
          <p class="text-xs text-gray-500 dark:text-gray-400 mt-1">
            Saved at {lastSaveTime.toLocaleTimeString()}
            {#if lastSavedProfileId && saveLocation === 'database'}
              • Profile ID: {lastSavedProfileId.substring(0, 8)}...
            {/if}
          </p>
        {/if}
      </div>

      <!-- Close Button -->
      {#if saveStatus !== 'saving'}
        <button 
          onclick={handleDismiss}
          class="flex-shrink-0 text-gray-400 hover:text-gray-600 dark:hover:text-gray-200 transition-colors"
          title="Dismiss"
          aria-label="Dismiss notification"
        >
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
          </svg>
        </button>
      {/if}
    </div>
  </div>
{/if}

<!-- Enhanced Validation Errors Section -->
{#if validationErrors.length > 0}
  <div class="mb-4 p-4 bg-red-50 dark:bg-red-900/20 border border-red-200 dark:border-red-800 rounded-lg">
    <div class="flex items-start gap-3">
      <ExclamationCircleOutline class="w-5 h-5 text-red-600 dark:text-red-400 mt-0.5 flex-shrink-0" />
      <div class="flex-1">
        <h3 class="text-sm font-medium text-red-800 dark:text-red-200 mb-2">
          Validation Issues ({validationErrors.length})
        </h3>
        <ul class="text-sm text-red-700 dark:text-red-300 space-y-1">
          {#each validationErrors as error, index}
            <li class="flex items-start gap-2">
              <span class="text-red-400 mt-1">•</span>
              <span>{error}</span>
            </li>
          {/each}
        </ul>
      </div>
    </div>
  </div>
{/if}