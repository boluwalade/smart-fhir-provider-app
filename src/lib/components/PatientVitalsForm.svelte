<script lang="ts">
    import axios from "axios"

    let temperature: number
    export let patient:string
    export let accessToken:string
    export let iss:string

    const postObservation = async (temperature:number) => {
        const temperatureResource = {
        "resourceType": "Observation",
        "status": "final",
        "category": [
            {
            "coding": [
                {
                "system": "http://terminology.hl7.org/CodeSystem/observation-category",
                "code": "vital-signs",
                "display": "Vital Signs"
                }
            ],
            "text": "Vital Signs"
            }
        ],
        "code": {
            "coding": [
            {
                "system": "http://loinc.org",
                "code": "8331-1"
            }
            ],
            "text": "Temperature Oral"
        },
        "subject": {
            "reference": `Patient/${patient}`
        },
        "effectiveDateTime": new Date().toISOString(),
        "valueQuantity": {
            "value": temperature,
            "unit": "degC",
            "system": "http://unitsofmeasure.org",
            "code": "Cel"
        }
    }
    console.log(temperatureResource)
    
    const response = await axios.post(`${iss}/Observation`,temperatureResource, {
        headers: {
            'Authorization': `Bearer ${accessToken}`
        }
    });
    console.log(response)
    }

</script>

<main>
    <div class="p-4 ">
        <h2 class="font-semibold">Create Patient Vitals</h2> 
        <form on:submit|preventDefault= {() => {postObservation(temperature)}}>
            <div class="mb-4">
                <label for="temperature" class="block text-sm font-medium text-gray-700">Temperature (°C)</label>
                <input type="number" id="temperature" bind:value={temperature} class="mt-1 border border-gray-800 rounded-md focus:ring-indigo-500 focus:border-indigo-500 block  shadow-sm sm:text-sm ">
            </div>
            <div class="mb-4">
                <button type="submit" class="bg-blue-800 text-white py-2 px-4 rounded">Submit</button>
            </div>
        </form>
    </div>

</main>