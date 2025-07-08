<!-- src/lib/components/profile/EvaluationCriteriaSection.svelte -->
<script lang="ts">
  import { Label, Select } from 'flowbite-svelte';

  // Props - bind to parent's criterion variables
  export let criterion1: string;
  export let criterion2: string;
  export let criterion3: string;

  // Evaluation criteria options
  const criteriaOptions = [
    { value: '', label: 'Select evaluation criteria' },
    { value: 'articleQuality', label: 'Article Quality' },
    { value: 'genderSenseAlignment', label: 'GenderSense Alignment' },
    { value: 'empoweringPositive', label: 'Empowering & Positive' },
    { value: 'currentUpToDate', label: 'Current & up to date' }
  ];
</script>

<!-- Evaluation Criteria Section -->
<div class="mb-6">
  <Label for="criterion1" class="block mb-2">
    Evaluation Criteria for Sources
  </Label>
  
  <Select 
    id="criterion1" 
    class="mb-6" 
    placeholder="Select evaluation criteria" 
    bind:value={criterion1}
  >
    {#each criteriaOptions as option}
      <option value={option.value}>{option.label}</option>
    {/each}
  </Select>
    
  <Label for="criterion2" class="block mb-2">
    Select
  </Label>
  <Select 
    id="criterion2" 
    class="mb-6" 
    placeholder="Select evaluation criteria" 
    bind:value={criterion2}
  >
    {#each criteriaOptions as option}
      <option value={option.value}>{option.label}</option>
    {/each}
  </Select>

  <Label for="criterion3" class="block mb-2">
    Select
  </Label>
  <Select 
    id="criterion3" 
    class="mb-6" 
    placeholder="Select evaluation criteria" 
    bind:value={criterion3}
  >
    {#each criteriaOptions as option}
      <option value={option.value}>{option.label}</option>
    {/each}
  </Select>
</div>