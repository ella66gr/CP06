<!-- src/routes/+page.svelte - Refactored CP06 Profile Editor -->

<script lang="ts">
  import { onMount } from 'svelte';
  import { ProfileManager } from '$lib/models/profileManager';
  import type { RSSFeed, CategoryTags } from '$lib/types';

  // Import refactored components
  import {
    StorageControls,
    CategoryTagsSection,
    ProfileFormSection,
    EvaluationCriteriaSection,
    SaveStatusBanner,
    ProfileActionButtons,
    RSSFeedManager
  } from '$lib/components/profile';

  // ===== PROFILE MANAGER & FORM VARIABLES =====
  
  // Initialize ProfileManager for structured profile handling
  let profileManager = new ProfileManager();

  // Form variables for user input
  let profile_name = '';
  let criterion1 = '';
  let criterion2 = '';
  let criterion3 = '';
  let profile_description = '';
  let tone_of_voice = '';
  let stepValue = 150;
  let newRssUrl = '';
  let newFeedName = '';

  // Category tags with binding
  let categoryTags: CategoryTags = {
    inTheNews: false,
    transHealth: false,
    genderSenseLatest: false,
    transitionCoaching: false,
    communityHighlights: false,
    transRights: false
  };

  // RSS Feed data with selection state
  let rssFeeds: RSSFeed[] = [];

  // ===== SAVE STATUS & UI STATE =====
  
  // Enhanced save status tracking
  let saveStatus: null | 'saving' | 'success' | 'error' = null;
  let validationErrors: string[] = [];
  let lastSaveTime: Date | null = null;
  let lastSavedProfileId: string | null = null;
  let saveLocation: 'local' | 'database' | null = null;
  let detailedSaveMessage: string = '';

  // Storage and client state
  const STORAGE_KEY = 'cp05_profile_data';
  let isClient = false;

  // ===== LIFECYCLE & INITIALIZATION =====
  
  onMount(() => {
    isClient = true;
    loadFromStorage();
  });

  // ===== PROFILE MANAGEMENT FUNCTIONS =====
  
  function updateProfileManager() {
    profileManager.updateFromFormData({
      profile_name,
      profile_description,
      tone_of_voice,
      criterion1,
      criterion2,
      criterion3,
      stepValue,
      categoryTags,
      rssFeeds
    });
  }

  function saveToStorage() {
    if (!isClient) return;
    
    updateProfileManager();
    const profileData = profileManager.getProfile();

    try {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(profileData));
      console.log('Profile data saved to localStorage', profileManager.getProfileSummary());
    } catch (error) {
      console.error('Failed to save to localStorage:', error);
      console.log('Failed to save profile data. Storage might be full.');
    }
  }

  function loadFromStorage() {
    if (!isClient) return;
    
    try {
      const saved = localStorage.getItem(STORAGE_KEY);
      if (saved) {
        const profileData = JSON.parse(saved);
        profileManager.loadFromJSON(profileData);
        
        // Update form fields from loaded profile
        const profile = profileManager.getProfile();
        profile_name = profile.profile_name;
        profile_description = profile.profile_description;
        tone_of_voice = profile.tone_of_voice;
        stepValue = profile.summaryLength;
        
        // Load evaluation criteria
        criterion1 = profile.evaluationCriteria[0] || '';
        criterion2 = profile.evaluationCriteria[1] || '';
        criterion3 = profile.evaluationCriteria[2] || '';
        
        // Load category tags
        categoryTags = { ...profile.categoryTags };
        
        // Load RSS feeds
        rssFeeds = profile.rssFeeds.map(feed => ({ ...feed, selected: false }));
        
        console.log('Profile data loaded from localStorage', profileManager.getProfileSummary());
      }
    } catch (error) {
      console.error('Failed to load from localStorage:', error);
    }
  }

  // ===== AUTO-SAVE FUNCTIONALITY =====
  
  let saveTimeout: ReturnType<typeof setTimeout> | undefined;
  function autoSave() {
    if (!isClient) return;
    clearTimeout(saveTimeout);
    saveTimeout = setTimeout(() => {
      saveToStorage();
    }, 1000);
  }

  // Reactive statements to trigger auto-save
  $: if (profile_name !== undefined) autoSave();
  $: if (profile_description !== undefined) autoSave();
  $: if (tone_of_voice !== undefined) autoSave();
  $: if (criterion1 !== undefined) autoSave();
  $: if (criterion2 !== undefined) autoSave();
  $: if (criterion3 !== undefined) autoSave();
  $: if (stepValue !== undefined) autoSave();
  $: if (categoryTags) autoSave();
  $: if (rssFeeds) autoSave();

  // ===== SAVE FUNCTIONS =====
  
  function saveProfile() {
    saveStatus = 'saving';
    saveLocation = 'local';
    detailedSaveMessage = 'Saving to local storage...';
    
    updateProfileManager();
    
    const validation = profileManager.validateForDatabase();
    validationErrors = validation.errors;
    
    if (!validation.isValid) {
      saveStatus = 'error';
      detailedSaveMessage = `Local save failed: ${validation.errors.length} validation error(s).`;
      return;
    }
    
    try {
      saveToStorage();
      
      saveStatus = 'success';
      lastSaveTime = new Date();
      saveLocation = 'local';
      
      const summary = profileManager.getProfileSummary();
      detailedSaveMessage = `Profile "${summary.name}" saved locally with ${summary.criteriaCount} criteria, ${summary.activeFeedsCount} active feeds.`;
      
      // Reset status after 3 seconds for local saves
      setTimeout(() => {
        saveStatus = null;
        detailedSaveMessage = '';
        saveLocation = null;
      }, 3000);
      
    } catch (error) {
      console.error('Failed to save profile locally:', error);
      saveStatus = 'error';
      saveLocation = 'local';
      detailedSaveMessage = 'Failed to save to local storage. Storage might be full.';
      validationErrors = ['Failed to save profile locally. Please try again.'];
    }
  }

  async function saveToDatabase(event: MouseEvent) {
    event.preventDefault();
    saveStatus = 'saving';
    saveLocation = 'database';
    detailedSaveMessage = 'Validating profile data...';
    
    updateProfileManager();
    
    const validation = profileManager.validateForDatabase();
    validationErrors = validation.errors;
    
    if (!validation.isValid) {
      saveStatus = 'error';
      detailedSaveMessage = `Validation failed: ${validation.errors.length} error(s) found.`;
      return;
    }

    try {
      detailedSaveMessage = 'Sending profile to database...';
      
      const databaseProfile = profileManager.prepareDatabaseProfile();
      
      const response = await fetch('/api/save-profile', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(databaseProfile)
      });

      const result = await response.json();
      
      if (response.ok && result.success) {
        console.log('Saved to DB with ID:', result.id);
        console.log('Profile Summary:', profileManager.getProfileSummary());
        
        saveStatus = 'success';
        lastSaveTime = new Date();
        lastSavedProfileId = result.id;
        
        const summary = profileManager.getProfileSummary();
        detailedSaveMessage = `Profile "${summary.name}" saved successfully to database with ID: ${result.id.substring(0, 8)}...`;
        
        // Also save to localStorage as backup
        saveToStorage();
        
        // Reset status after 5 seconds for database saves
        setTimeout(() => {
          saveStatus = null;
          detailedSaveMessage = '';
          saveLocation = null;
        }, 5000);
      } else {
        throw new Error(result.error || result.details || 'Failed to save profile to database');
      }
    } catch (error) {
      console.error('Error saving profile to DB:', error);
      saveStatus = 'error';
      saveLocation = 'database';
      
      if (error instanceof Error) {
        detailedSaveMessage = `Database save failed: ${error.message}`;
        validationErrors = [error.message];
      } else {
        detailedSaveMessage = 'Unknown error occurred while saving to database.';
        validationErrors = ['Failed to save to database. Please try again.'];
      }
    }
  }

  // ===== STORAGE EVENT HANDLERS =====
  
  function handleExport() {
    updateProfileManager();
    
    const defaultFilename = profile_name 
      ? `${profile_name.toLowerCase().replace(/[^a-z0-9]/g, '-')}-profile`
      : 'cp05-profile';
    
    const userFilename = prompt('Enter filename for export:', defaultFilename);
    if (userFilename === null) return;
    
    const cleanFilename = userFilename.trim() || defaultFilename;
    const finalFilename = cleanFilename.endsWith('.json') 
      ? cleanFilename 
      : `${cleanFilename}.json`;

    const jsonString = profileManager.exportAsJSON();
    const blob = new Blob([jsonString], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    
    const a = document.createElement('a');
    a.href = url;
    a.download = finalFilename;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
    
    console.log('Profile exported as:', finalFilename);
  }

  function handleImport(event: CustomEvent<{ file: File }>) {
    const file = event.detail.file;
    if (!file) return;

    const reader = new FileReader();
    reader.onload = (e) => {
      try {
        const importedData = JSON.parse((e.target as FileReader).result as string);
        profileManager.loadFromJSON(importedData);
        
        // Update form fields from imported profile
        const profile = profileManager.getProfile();
        profile_name = profile.profile_name;
        profile_description = profile.profile_description;
        tone_of_voice = profile.tone_of_voice;
        stepValue = profile.summaryLength;
        
        // Load evaluation criteria
        criterion1 = profile.evaluationCriteria[0] || '';
        criterion2 = profile.evaluationCriteria[1] || '';
        criterion3 = profile.evaluationCriteria[2] || '';
        
        // Load category tags
        categoryTags = { ...profile.categoryTags };
        
        // Load RSS feeds
        rssFeeds = profile.rssFeeds.map(feed => ({ ...feed, selected: false }));
        
        saveToStorage();
        alert('Profile imported successfully!');
        console.log('Profile imported from JSON');
      } catch (error) {
        console.error('Failed to import profile:', error);
        alert('Failed to import profile. Please check the file format.');
      }
    };
    reader.readAsText(file);
  }

  function handleClearAll() {
    if (!isClient) return;
    
    if (confirm('Are you sure you want to clear all profile data? This cannot be undone.')) {
      localStorage.removeItem(STORAGE_KEY);
      
      // Reset ProfileManager
      profileManager = new ProfileManager();
      
      // Reset all form fields
      profile_name = '';
      profile_description = '';
      tone_of_voice = '';
      criterion1 = '';
      criterion2 = '';
      criterion3 = '';
      stepValue = 150;
      categoryTags = {
        inTheNews: false,
        transHealth: false,
        genderSenseLatest: false,
        transitionCoaching: false,
        communityHighlights: false,
        transRights: false
      };
      
      rssFeeds = [];
      
      console.log('All profile data cleared');
    }
  }

  // ===== RSS FEED EVENT HANDLERS =====
  
  function handleAddFeed(event: CustomEvent<{ name: string; url: string }>) {
    const { name, url } = event.detail;
    
    const nextId = rssFeeds.length > 0 ? Math.max(...rssFeeds.map(feed => feed.id)) + 1 : 1;

    const newFeed: RSSFeed = {
      id: nextId,
      name,
      url,
      status: "active",
      selected: false
    };

    rssFeeds = [...rssFeeds, newFeed];
    
    newFeedName = '';
    newRssUrl = '';
    
    console.log('Added new RSS feed:', newFeed);
  }

  function handleToggleSelectedFeeds(event: CustomEvent<{ feedIds: number[] }>) {
    const { feedIds } = event.detail;
    
    rssFeeds = rssFeeds.map(feed => {
      if (feedIds.includes(feed.id)) {
        return {
          ...feed,
          status: feed.status === 'active' ? 'paused' : 'active'
        };
      }
      return feed;
    });
    
    console.log('Toggled status for selected feeds');
  }

  function handleDeleteSelectedFeeds(event: CustomEvent<{ feedIds: number[] }>) {
    const { feedIds } = event.detail;
    
    rssFeeds = rssFeeds.filter(feed => !feedIds.includes(feed.id));
    console.log('Deleted selected feeds');
  }

  function handleUpdateFeeds(event: CustomEvent<{ feeds: RSSFeed[] }>) {
    rssFeeds = event.detail.feeds;
  }

  // ===== OTHER EVENT HANDLERS =====
  
  function handleCancel() {
    if (confirm('Are you sure you want to cancel? All unsaved changes will be lost.')) {
      loadFromStorage();
      saveStatus = null;
      validationErrors = [];
      detailedSaveMessage = '';
      saveLocation = null;
    }
  }

  function handleDismissStatus() {
    saveStatus = null;
    detailedSaveMessage = '';
    saveLocation = null;
  }

  // ===== REACTIVE PROFILE SYNC =====
  
  $: {
    if (isClient) {
      updateProfileManager();
    }
  }

  // ===== COMPUTED VALUES =====
  
  $: profileSummary = isClient ? profileManager.getProfileSummary() : null;
</script>

<!-- ===== PAGE TITLE ===== -->
<svelte:head>
    <title>CP06 Profile Editor</title>
</svelte:head>

<!-- ===== MAIN CONTAINER ===== -->
<div class="min-h-screen bg-green-100 dark:bg-gray-900">

  <!-- MAIN PANEL - Full width -->
  <div class="bg-white dark:bg-gray-800 border-l border-r border-b border-gray-200 dark:border-gray-700 mt-0">

    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-6">

      <!-- ===== STORAGE CONTROLS ===== -->
      <StorageControls 
        {profile_name}
        on:export={handleExport}
        on:import={handleImport}
        on:clearAll={handleClearAll}
      />

      <!-- ===== SAVE STATUS SECTION ===== -->
      <div class="mb-6">
        <SaveStatusBanner 
          {saveStatus}
          {validationErrors}
          {lastSaveTime}
          {lastSavedProfileId}
          {saveLocation}
          {detailedSaveMessage}
          on:dismiss={handleDismissStatus}
        />

        <ProfileActionButtons 
          {saveStatus}
          {saveLocation}
          {profile_name}
          {profileSummary}
          on:saveToDatabase={saveToDatabase}
          on:saveLocally={saveProfile}
          on:cancel={handleCancel}
        />
      </div>

      <!-- ===== MAIN FORM GRID ===== -->
      <form>
        <div class="mb-6 grid gap-8 md:grid-cols-2">
          
          <!-- Left Column -->
          <div>
            <ProfileFormSection 
              bind:profile_name
              bind:profile_description
              bind:tone_of_voice
              bind:stepValue
            />
          </div>

          <!-- Right Column -->
          <div>
            <EvaluationCriteriaSection 
              bind:criterion1
              bind:criterion2
              bind:criterion3
            />

            <CategoryTagsSection 
              bind:categoryTags
            />
          </div>

        </div>
      </form>

      <!-- ===== RSS FEEDS SECTION ===== -->
      <RSSFeedManager 
        bind:rssFeeds
        bind:newRssUrl
        bind:newFeedName
        on:addFeed={handleAddFeed}
        on:toggleSelectedFeeds={handleToggleSelectedFeeds}
        on:deleteSelectedFeeds={handleDeleteSelectedFeeds}
        on:updateFeeds={handleUpdateFeeds}
      />

    </div>

  </div>

</div>