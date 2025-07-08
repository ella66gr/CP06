<!-- src/lib/components/profile/RSSFeedManager.svelte -->
<script lang="ts">
  import { 
    Button,
    Dropdown,
    DropdownItem,
    Input,
    Label,
    Checkbox,
    Badge
  } from 'flowbite-svelte';
  import {
    Table,
    TableBody,
    TableBodyCell,
    TableBodyRow,
    TableHead,
    TableHeadCell
  } from 'flowbite-svelte';
  import { AlignJustifyOutline, ChevronDownOutline } from 'flowbite-svelte-icons';
  import { createEventDispatcher } from 'svelte';
  import type { RSSFeed } from '$lib/types';

  // Props
  export let rssFeeds: RSSFeed[];
  export let newRssUrl: string = '';
  export let newFeedName: string = '';

  // Event dispatcher
  const dispatch = createEventDispatcher<{
    addFeed: { name: string; url: string };
    toggleSelectedFeeds: { feedIds: number[] };
    deleteSelectedFeeds: { feedIds: number[] };
    updateFeeds: { feeds: RSSFeed[] };
  }>();

  // Reactive calculations
  $: selectedFeeds = rssFeeds.filter(feed => feed.selected);
  $: hasSelectedFeeds = selectedFeeds.length > 0;
  $: allSelected = rssFeeds.length > 0 && rssFeeds.every(feed => feed.selected);

  // Event handlers
  function handleAddNewSource() {
    if (!newFeedName.trim()) {
      alert('Please enter a feed name');
      return;
    }

    if (!newRssUrl.trim()) {
      alert('Please enter a valid RSS feed URL');
      return;
    }

    try {
      new URL(newRssUrl);
    } catch {
      alert('Please enter a valid URL');
      return;
    }

    const isDuplicateUrl = rssFeeds.some(feed => feed.url === newRssUrl.trim());
    if (isDuplicateUrl) {
      alert('This RSS feed URL already exists');
      return;
    }

    const isDuplicateName = rssFeeds.some(feed => 
      feed.name.toLowerCase() === newFeedName.trim().toLowerCase()
    );
    if (isDuplicateName) {
      alert('This feed name already exists');
      return;
    }

    dispatch('addFeed', {
      name: newFeedName.trim(),
      url: newRssUrl.trim()
    });
  }

  function handleToggleSelectedFeeds() {
    const selectedFeedIds = selectedFeeds.map(feed => feed.id);
    dispatch('toggleSelectedFeeds', { feedIds: selectedFeedIds });
  }

  function handleDeleteSelectedFeeds() {
    if (confirm(`Are you sure you want to delete ${selectedFeeds.length} selected feed(s)?`)) {
      const selectedFeedIds = selectedFeeds.map(feed => feed.id);
      dispatch('deleteSelectedFeeds', { feedIds: selectedFeedIds });
    }
  }

  function handleSelectAll() {
    const shouldSelectAll = !allSelected;
    const updatedFeeds = rssFeeds.map(feed => ({
      ...feed,
      selected: shouldSelectAll
    }));
    dispatch('updateFeeds', { feeds: updatedFeeds });
  }

  function handleFeedSelectionChange(feedId: number, selected: boolean) {
    const updatedFeeds = rssFeeds.map(feed => 
      feed.id === feedId ? { ...feed, selected } : feed
    );
    dispatch('updateFeeds', { feeds: updatedFeeds });
  }
</script>

<!-- RSS Feeds Section -->
<div class="mb-6">
  <h2 class="text-lg font-semibold mb-4">RSS Feeds</h2>
  <p class="text-gray-600 dark:text-gray-300 mb-4">Add RSS feeds to be used for content generation.</p>

  <!-- Add New Feed Form -->
  <div class="mb-6 grid gap-8 md:grid-cols-3">
    <!-- Col 1 - Feed Name -->
    <div class="mb-0 col-span-1">
      <Label for="feed_name" class="block mb-2">Feed Name</Label>
      <Input 
        type="text" 
        id="feed_name" 
        placeholder="Example News" 
        bind:value={newFeedName}
        class="w-full mb-6"
      />

      <Button 
        type="button"
        color="primary" 
        onclick={handleAddNewSource}
      >
        Add New Source
      </Button>
    </div>

    <!-- Col 2 - RSS URL -->
    <div class="mb-4 col-span-2">
      <Label for="rss_feed" class="block mb-2">RSS Feed URL</Label>
      <Input 
        type="url" 
        id="rss_feed" 
        placeholder="https://example.com/rss" 
        bind:value={newRssUrl}
        class="w-full mb-4"
      />
    </div>

    <!-- Col 3 - Empty -->
    <div></div>
  </div>

  <!-- RSS Feeds Display Table -->
  <div class="mt-6 overflow-x-auto">
    <Table shadow>
      <TableHead>
        <TableHeadCell>Source Name</TableHeadCell>
        <TableHeadCell>
          <div class="flex items-center justify-between">
            <span>RSS Feed URL</span>
            {#if hasSelectedFeeds}
              <span class="text-xs bg-blue-100 text-blue-800 dark:bg-blue-900/30 dark:text-blue-300 px-2 py-1 rounded-full ml-2">
                {selectedFeeds.length} selected
              </span>
            {/if}
          </div>
        </TableHeadCell>
        <TableHeadCell>Status</TableHeadCell>
        <TableHeadCell>
          <div class="flex items-center gap-2">
            <!-- Select All Checkbox -->
            <Checkbox 
              checked={allSelected}
              onclick={handleSelectAll}
              title="Select All"
            />
            
            <!-- Actions Dropdown -->
            <div class="relative">
              <Button 
                color="light" 
                class="!p-2"
                disabled={!hasSelectedFeeds}
                id="dropdown-button"
              >
                <AlignJustifyOutline class="w-4 h-4" />
                <ChevronDownOutline class="w-3 h-3 ml-1" />
              </Button>
              
              <Dropdown triggeredBy="#dropdown-button" class="w-48" placement="top">
                <DropdownItem 
                  onclick={handleToggleSelectedFeeds}
                  disabled={!hasSelectedFeeds}
                  class="flex items-center gap-2 dark:text-gray-300"
                >
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7h12m0 0l-4-4m4 4l-4 4m0 6H4m0 0l4 4m-4-4l4-4"/>
                  </svg>
                  Toggle Active/Pause
                </DropdownItem>
                <DropdownItem 
                  onclick={handleDeleteSelectedFeeds}
                  disabled={!hasSelectedFeeds}
                  class="flex items-center gap-2 text-red-600 dark:text-red-400"
                >
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"/>
                  </svg>
                  Delete Selected
                </DropdownItem>
              </Dropdown>
            </div>
          </div>
        </TableHeadCell>
      </TableHead>
      
      <TableBody>
        {#each rssFeeds as feed (feed.id)}
          <TableBodyRow>
            <TableBodyCell>{feed.name}</TableBodyCell>
            <TableBodyCell class="font-mono text-sm">{feed.url}</TableBodyCell>
            <TableBodyCell>
              <Badge color={feed.status === 'active' ? 'green' : 'red'}>
                {feed.status === 'active' ? 'Active' : 'Paused'}
              </Badge>
            </TableBodyCell>
            <TableBodyCell>
              <Checkbox 
                checked={feed.selected}
                onchange={(e) => handleFeedSelectionChange(feed.id, e.target.checked)}
              />
            </TableBodyCell>
          </TableBodyRow>
        {/each}
      </TableBody>
    </Table>
  </div>
</div>