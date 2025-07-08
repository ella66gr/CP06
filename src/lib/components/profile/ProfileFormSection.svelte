<!-- src/lib/components/profile/ProfileFormSection.svelte -->
<script lang="ts">
  import { Input, Textarea, Label, Select, Range } from 'flowbite-svelte';

  // Props - bind to parent's form variables
  export let profile_name: string;
  export let profile_description: string;
  export let tone_of_voice: string;
  export let stepValue: number;
</script>

<!-- Profile Form Section -->
<div class="mb-6">
  <Label for="profile_name" class="mb-2">
    Profile Name
  </Label>
  <Input 
    type="text" 
    id="profile_name" 
    placeholder="Name" 
    required 
    class="mb-6" 
    bind:value={profile_name}
  />

  <Label for="profile_description" class="mb-2">
    Profile Description
  </Label>
  <Textarea
    id="profile_description"
    class="mb-6 text-md font-medium text-gray-900 dark:text-white"
    placeholder="Optional profile description and purpose …"
    bind:value={profile_description} 
    rows={5}
  />

  <Label for="tone_of_voice" class="mb-2 mt-0">
    Tone of Voice
  </Label>
  <Select
    id="tone_of_voice"
    class="block mb-6 text-sm font-medium text-gray-900 dark:text-white"
    placeholder="Select tone of voice"
    bind:value={tone_of_voice}
  >
    <option value="">Select tone of voice</option>
    <option value="professional_supportive">Professional / Supportive</option>
    <option value="empowering_activist">Empowering / Activist</option>
    <option value="community_caring">Community & Caring</option>
  </Select>

  <Label for="range-steps" class="mb-2 mt-8">
    Summary Text Length (max 400)
  </Label>
  <Range 
    id="range-steps" 
    min="0" 
    max="400" 
    bind:value={stepValue} 
    step="10" 
  />
  <p class="mt-2">Value: {stepValue}</p>
</div>