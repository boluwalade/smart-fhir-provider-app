<script lang="ts">
    import { CLIENT_ID, SCOPES, REDIRECT_URL, STATE } from '../FhirStores';
    import axios from 'axios';
    import pkceChallenge from "pkce-challenge";
    import { onMount } from 'svelte';
    import PatientBanner from './PatientBanner.svelte';
    import PatientVitalsRead from './PatientVitalsRead.svelte';
    import PatientVitalsForm from './PatientVitalsForm.svelte';

interface tokenResponse {
    "access_token": string,
    "patient": string,
    "scope": string,
    "need_patient_banner": Boolean,
    "id_token": String,
    "smart_style_url": String,
    "encounter": String,
    "token_type": String,
    "expires_in": number
}
    let parsedToken:tokenResponse | undefined ;
    const LOCAL_STORAGE_TOKEN_KEY = 'tokenResponse';



    const generateCodeVerifier = async () => {
        const {code_challenge,code_verifier} = await pkceChallenge();
        localStorage.setItem('code_verifier', code_verifier);
        return code_challenge;
    }
      // Function to construct authorization URL with PKCE parameters
    const getRedirectUrl = (
        codeChallenge: string,
        authorization_endpoint: string,
        iss: string,
        launch: string,
        ) => {
        const authorizeURL = new URL(authorization_endpoint);
        authorizeURL.searchParams.set('aud', iss);
        authorizeURL.searchParams.set('launch', launch);
        authorizeURL.searchParams.set('client_id', CLIENT_ID);
        authorizeURL.searchParams.set('scope', SCOPES);
        authorizeURL.searchParams.set('redirect_uri', REDIRECT_URL);
        authorizeURL.searchParams.set('response_type', 'code');
        authorizeURL.searchParams.set('state', STATE);
        authorizeURL.searchParams.set('code_challenge', codeChallenge);
        authorizeURL.searchParams.set('code_challenge_method', 'S256');
        
        return authorizeURL.href; // Return authorization URL (contains authorization code and state(if set))
    };

    //function to get token from local storage
    const postAccessTokenRequest = async (code: string) => {
        const token_endpoint = localStorage.getItem('token_endpoint') as string
        const searchParams = new URLSearchParams();
        searchParams.set('code', code);
        searchParams.set('grant_type', 'authorization_code');
        searchParams.set('redirect_uri', REDIRECT_URL);
        searchParams.set('client_id', CLIENT_ID);
        const code_verifier = localStorage.getItem('code_verifier') as string;
        searchParams.set('code_verifier', code_verifier);
        const response = await axios.post(token_endpoint,searchParams.toString(), {
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded',
                'accept': 'application/json'
            }
         });
         return response.data;
    }



onMount(async () => {

    //get iss from url
    const url = new URL(window.location.href);
    const launch = url.searchParams.get('launch') as string;
    const iss = url.searchParams.get('iss') as string
    const code = url.searchParams.get('code') as string;
    const state = url.searchParams.get('state') as string;

    
    if(iss && launch) {
        //save iss to local storage
        localStorage.setItem('iss', iss);
        //get well-known configuration
        const smartConfig = await axios.get(`${iss}/.well-known/smart-configuration`);
        const smartConfigData = smartConfig.data;
        const authorization_endpoint = smartConfigData.authorization_endpoint;
        const token_endpoint = smartConfigData.token_endpoint;
        localStorage.setItem('token_endpoint', token_endpoint);
         // //get code challenge
        const codeChallenge = await generateCodeVerifier();
        //get redirect url
        const redirect_url = getRedirectUrl(codeChallenge, authorization_endpoint, iss, launch);  
        window.location.href = redirect_url; // Redirect user to authorization URL
        
    }
    //get token from local storage
    // const storedTokenResponse = localStorage.getItem(LOCAL_STORAGE_TOKEN_KEY);
    // if(storedTokenResponse) {
    //     parsedToken = JSON.parse(storedTokenResponse as string);
    //     //validate token expiration
    //     const currentTime = Math.floor(Date.now() / 1000); // Current time in seconds since epoch
    //     const expiryTime = parsedToken.expires_in + currentTime;

    //     // if (expiryTime > currentTime) {
    //     //     console.log("Token has expired.");
    //     //     localStorage.removeItem(LOCAL_STORAGE_TOKEN_KEY); // Remove expired token
    //     //     // Optionally, redirect to login or refresh the token
    //     // } else {
    //     //     console.log("Token is still valid.");
    //     // }  
    // }
    
    if (code && state === STATE) {
        const tokenResponse = await postAccessTokenRequest(code);
        parsedToken = tokenResponse
        localStorage.setItem(LOCAL_STORAGE_TOKEN_KEY, JSON.stringify(tokenResponse));
        
    }
});

const iss = localStorage.getItem('iss') as string;


</script>

<main>
{#if !parsedToken}
    Loading...
{:else}
    {#if parsedToken?.need_patient_banner}
    <PatientBanner accessToken={parsedToken.access_token} iss={iss} patient={parsedToken.patient}> </PatientBanner>
    {/if}
    <PatientVitalsForm accessToken = {parsedToken.access_token} iss={iss} patient= {parsedToken.patient}></PatientVitalsForm>
    <PatientVitalsRead accessToken={parsedToken.access_token} iss={iss} patient={parsedToken.patient}> </PatientVitalsRead>
{/if}
</main>

<style>

</style>
