<script lang="ts">
    import type { Patient } from "fhir/r4";
    import { onMount } from "svelte";
    import axios from 'axios';

    let patientResource: Patient | undefined = undefined;
    export let accessToken: string;
    export let iss: string;
    export let patient: string;

    const getPatientBanner = async (accessToken: string, iss: string, patient: string) => {
        const response = await axios.get<Patient>(`${iss}/Patient/${patient}`, {
            headers: {
                "Authorization": `Bearer ${accessToken}`,
                'Accept': 'application/json'
            },
        });
        return response.data;;
    }

    onMount(async () => {
        patientResource = await getPatientBanner(accessToken, iss, patient);
        console.log(patientResource);
    });
</script>

<main>
    {#if !patientResource}
        <h1>Loading...</h1>
    {:else}
        <div class="bg-blue-100 p-4 rounded-lg shadow-md">
            <h2 class="text-lg font-bold text-blue-800">Patient Details</h2>
            {#if patientResource?.name}
                <p><strong>Name:</strong> {patientResource.name[0]?.text || 'N/A'}</p>
            {/if}
            <p><strong>Gender:</strong> {patientResource?.gender || 'N/A'}</p>
            <p><strong>Birth Date:</strong> {patientResource?.birthDate || 'N/A'}</p>
            {#if patientResource?.address && patientResource.address.length > 0}
                <p><strong>Address:</strong> {patientResource.address[0]?.line?.join(", ") || 'N/A'}, {patientResource.address[0]?.city || 'N/A'}, {patientResource.address[0]?.state || 'N/A'} {patientResource.address[0]?.postalCode || 'N/A'}</p>
            {/if}
        </div>
    {/if}
</main>

<style>

</style>

