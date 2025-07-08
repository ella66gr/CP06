<!-- src/lib/components/profile/CategoryTagsSection.svelte -->
<script lang="ts">
  import { Card, Checkbox, Helper } from 'flowbite-svelte';
  import type { CategoryTags } from '$lib/types';

  // Props - bind to parent's categoryTags object
  export let categoryTags: CategoryTags;
</script>

<!-- Category Tags Section -->
<div class="mb-6">
  <h5 class="mt-8 mb-2 text-sm">Available Category Tags (e.g. newsletter sections)</h5>
  <Card class="p-4 sm:p-6 md:p-8 min-w-full shadow-none">
    
    <Checkbox aria-describedby="news-helper" bind:checked={categoryTags.inTheNews}>
      In the News
    </Checkbox>
    <Helper id="news-helper" class="ps-7 mb-4">
      News items of general Trans+ interest
    </Helper>

    <Checkbox aria-describedby="health-helper" bind:checked={categoryTags.transHealth}>
      Trans+ Health
    </Checkbox>
    <Helper id="health-helper" class="ps-7 mb-4">
      Trans+ health tips and info
    </Helper>

    <Checkbox aria-describedby="latest-helper" bind:checked={categoryTags.genderSenseLatest}>
      GenderSense Latest
    </Checkbox>
    <Helper id="latest-helper" class="ps-7 mb-4">
      Updates on what is going on at GenderSense
    </Helper>

    <Checkbox aria-describedby="coaching-helper" bind:checked={categoryTags.transitionCoaching}>
      Transition Coaching Tips
    </Checkbox>
    <Helper id="coaching-helper" class="ps-7 mb-4">
      Hints and tips on making life easier
    </Helper>

    <Checkbox aria-describedby="community-helper" bind:checked={categoryTags.communityHighlights}>
      GenderSense Community Highlights
    </Checkbox>
    <Helper id="community-helper" class="ps-7 mb-4">
      Items from the GenderSense online community
    </Helper>

    <Checkbox aria-describedby="rights-helper" bind:checked={categoryTags.transRights}>
      Trans+ Issues & Rights Updates
    </Checkbox>
    <Helper id="rights-helper" class="ps-7 mb-4">
      Community issues
    </Helper>

  </Card>
</div>