<!-- src/lib/components/profile/ProfileActionButtons.svelte -->
<script lang="ts">
  import { Button } from 'flowbite-svelte';
  import { createEventDispatcher } from 'svelte';

  // Props
  export let saveStatus: null | 'saving' | 'success' | 'error' = null;
  export let saveLocation: 'local' | 'database' | null = null;
  export let profile_name: string = '';
  export let profileSummary: {
    name: string;
    criteriaCount: number;
    activeFeedsCount: number;
    selectedCategoriesCount: number;
    isComplete: boolean;
  } | null = null;

  // Event dispatcher
  const dispatch = createEventDispatcher<{
    saveToDatabase: MouseEvent;
    saveLocally: void;
    cancel: void;
  }>();

  function handleSaveToDatabase(event: MouseEvent) {
    dispatch('saveToDatabase', event);
  }

  function handleSaveLocally() {
    dispatch('saveLocally');
  }

  function handleCancel() {
    dispatch('cancel');
  }
</script>

<!-- Action Buttons Section -->
<div class="flex flex-wrap items-center justify-between gap-4">
  <!-- Left side - Profile Summary (when not saving) -->
  <div class="flex items-center gap-4">
    {#if !saveStatus && profile_name}
      <div class="text-sm text-gray-600 dark:text-gray-300">
        <span class="font-medium">{profile_name}</span>
        {#if profileSummary}
          <span class="text-xs ml-2">
            {profileSummary.criteriaCount} criteria • {profileSummary.activeFeedsCount} feeds • {profileSummary.selectedCategoriesCount} categories
            {#if profileSummary.isComplete}
              <span class="text-green-600 dark:text-green-400 ml-1">✓ Complete</span>
            {:else}
              <span class="text-amber-600 dark:text-amber-400 ml-1">⚠ Incomplete</span>
            {/if}
          </span>
        {/if}
      </div>
    {/if}
  </div>

  <!-- Right side - Action buttons -->
  <div class="flex items-center gap-3">
    <Button 
      color="alternative" 
      onclick={handleCancel} 
      disabled={saveStatus === 'saving'}
    >
      Cancel
    </Button>
    
    <Button 
      color="primary" 
      onclick={handleSaveToDatabase} 
      disabled={saveStatus === 'saving'}
      class="relative"
    >
      {#if saveStatus === 'saving' && saveLocation === 'database'}
        <svg class="animate-spin h-4 w-4 mr-2" fill="none" viewBox="0 0 24 24">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
          <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
        </svg>
      {/if}
      Save to Database
    </Button>
    
    <Button 
      color="green" 
      onclick={handleSaveLocally} 
      disabled={saveStatus === 'saving'}
      class="relative"
    >
      {#if saveStatus === 'saving' && saveLocation === 'local'}
        <svg class="animate-spin h-4 w-4 mr-2" fill="none" viewBox="0 0 24 24">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
          <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
        </svg>
      {/if}
      Save Locally
    </Button>
  </div>
</div>