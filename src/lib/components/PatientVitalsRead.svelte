<script lang="ts">

    import axios from 'axios';
    import type { Bundle, Observation, OperationOutcome } from 'fhir/r4';
    import { onMount } from 'svelte';


    export let patient: any;
    export let iss: string;
    export let accessToken: string;
    let observations: any;

    const getObservations = async (iss: string,patient: string,accessToken: string) => {
        const response = await axios.get<Bundle<Observation|OperationOutcome>>(`${iss}/Observation?category=vital-signs&patient=${patient}`, {
            headers: {
                'Authorization': `Bearer ${accessToken}`,
                'Accept': 'application/json'
            }
        });
        return response.data;
    }

    onMount(async () => {
        if(patient && iss && accessToken){
            const response = await getObservations(iss,patient,accessToken);
            observations = response;
            console.log(observations);
        }
    })

</script>

<main>
    {#if !observations}
        <p class="text-grey-500">Loading...</p>
    {:else}
    <div class="vitals-container space-y-4">
            {#each observations.entry as {resource}}
                {#if resource.resourceType === 'Observation'}
                    <div class="vital-item bg-gray-100 p-4 rounded-lg shadow">
                        <h3 class="text-lg font-bold text-blue-800">{resource.code?.text || 'Unknown Vital'}</h3>
                        <p><strong>Value:</strong> {resource.valueQuantity?.value} {resource.valueQuantity?.unit}</p>
                        <p><strong>Date:</strong> {resource.effectiveDateTime}</p>
                    </div>
                {/if}
            {/each}
    </div>
    {/if}
</main>

<style>

</style>
