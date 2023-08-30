<script>
   import { Input, Label} from 'flowbite-svelte';
	import 'flowbite/dist/flowbite.css';
	import { Select} from 'flowbite-svelte';
	import { onMount } from 'svelte';	
	import { MultiSelect } from 'flowbite-svelte';
	import "intl-tel-input/build/css/intlTelInput.css";
	import intlTelInput from "intl-tel-input";
	
   
 
	export	let currencyOptions = [];
	export  let selectedTimezone;
    export  let reactiveTimezoneOptions = [];
	export	let countryOptions = '';
    export  let countries = [];
    export let selectedCodes = [];
	export	let imageUrl = null;
	export  let phone='';
	export	let iti;
	
	
	
	
	
	
   
	
	export let data;
	
    let name = data.profiledata.name;
	let  website = data.profiledata.website;
	let cityName=data.profiledata.address.cityName;
	let  currencyCode = data.profiledata.currencyCode.code;
	let  preferredDateformat = data.profiledata.preferredDateFormat; 
	let timeZone = data.profiledata.timeZone;
	let preferredCountryCode = data.profiledata.mobilePreferences.preferredCountryCode;
	let preferredCountryCode1 = preferredCountryCode.toUpperCase();
	let  preferredCountries = data.profiledata.mobilePreferences.preferredCountries;
	let selectedLabels = preferredCountries.split(",");
	console.log(preferredCountryCode1);
	console.log(currencyCode);
	console.log(selectedLabels);
	console.log(cityName);
  
    async function updateData() {
      const updatedData = {
        ...data.profiledata,
         name,
		 website,
		cityName,
				
		phone,
		 currencyCode: {

					code: currencyCode,
					},
		 preferredDateformat,
		 timeZone,
		 preferredCountries: selectedLabels,
		 preferredCountryCode: preferredCountryCode1.toLowerCase(),
      };
  
      const response = await fetch("https://api.recruitly.io/api/business/profile/save?apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77", {
        method: 'POST', 
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(updatedData),
      });
  
    
    }
	
	onMount(async () => {
  try {
    const currencyResponse = await fetch('https://api.recruitly.io/api/lookup/currency?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA');
    const currencyData = await currencyResponse.json();

    if (Array.isArray(currencyData.data)) {
      currencyOptions = currencyData.data.map((currency) => ({
        value: currency.code,
        label: currency.name,
      }));
	
    } else {
      console.error('Invalid currency data format:', currencyData);
    }

    const timezoneResponse = await fetch('https://api.recruitly.io/api/lookup/timezone?apiKey=TEST27306FA00E70A0F94569923CD689CA9BE6CA');
    const timezoneData = await timezoneResponse.json();

    if (Array.isArray(timezoneData.data)) {
      reactiveTimezoneOptions = timezoneData.data.map((timezone, index) => ({
        ...timezone,
        id: index + 1,
      }));
      selectedTimezone = reactiveTimezoneOptions.length > 0 ? reactiveTimezoneOptions[0].code : null;
    } else {
      console.error('Invalid timezone data format:', timezoneData);
    }

   
  } catch (error) {
    console.error('Error fetching options:', error);
  }
});

const fetchCountryData = async () => {
      try {
      const response = await fetch(
        "https://api.recruitly.io/api/lookup/countries?apiKey=TEST69513C4B379BD5594CD0AAC9ECA436CA2C83" 
             );
   const responseData = await response.json();
     if (Array.isArray(responseData.data)) {
          countryOptions = responseData.data.map((country) => ({
              value: country.code,
              label: country.name,
              }));
		console.log("Country API response:", responseData);
		     } else {
		console.error("Invalid country data format:", responseData);
		    }
		      } catch (error) {
		         console.error("Error fetching country options:", error);

				}
          };


	
		
async function fetchCountries() {
try {

	const response = await fetch(

		"https://api.recruitly.io/api/lookup/countries?apiKey=TEST69513C4B379BD5594CD0AAC9ECA436CA2C83"

	);

	const responseData = await response.json();



	if (Array.isArray(responseData.data)) {

		countries = responseData.data.map((country) => ({

			name: country.name,

			value: country.code.toLowerCase(),

		}));

		selectedCodes = countries

			.filter((country) => selectedLabels.includes(country.value))

			.map((selectedCountry) => selectedCountry.value);

	} else {

		console.error("Invalid API response format:", responseData);

	}

} catch (error) {

	console.error("Error fetching data:", error);

}

}



function handleSelection(event) {

if (event && event.detail && event.detail.value) {

	selectedCodes = event.detail.value.map((selectedCountry) =>

		selectedCountry.value.toLowerCase()

	);

}

}


async function fetchImage() {
  const apiKey = "TEST45684CB2A93F41FC40869DC739BD4D126D77";
  const uniqueParam = Date.now(); // Generate a unique parameter using the current timestamp
  const apiUrl = `https://api.recruitly.io/api/business/profile?apiKey=${apiKey}&timestamp=${uniqueParam}`;

  try {
    const response = await fetch(apiUrl);
    const data = await response.json();
    // Access the imageUrl from the appLogo object
    imageUrl = data.logo.url;
	
	
  } catch (error) {
    console.error("Error fetching image:", error);
  }
}


	// Function to handle the logo click and trigger image upload
	function handleLogoClick() {
	  const inputElement = document.createElement("input");
	  inputElement.type = "file";
	  inputElement.accept = "image/*";
	  inputElement.addEventListener("change", uploadImage);
	  inputElement.click();
	}
  
	// Function to handle image upload
	async function uploadImage(event) {
  const apiKey = "TEST45684CB2A93F41FC40869DC739BD4D126D77";
  const uploadUrl = `https://api.recruitly.io/api/image/upload?apiKey=${apiKey}`;
  const formData = new FormData();
  formData.append("file", event.target.files[0]);
  formData.append("profilePic", "true");
  formData.append("type", "TENANT");

  try {
    const response = await fetch(uploadUrl, {
      method: "POST",
      headers: {
        Cookie: "SESSION=NDU1ZDRjNmUtNDg1ZC00NjVhLWJhNmItN2NlZmE4NzYxMWRm",
        "Cache-Control": "no-cache, no-store, must-revalidate", // Add the Cache-Control header
      },
      body: formData,
    });
    const data = await response.json();
    // The uploaded image URL should be available in 'data.url'
    imageUrl = data.logo.url;
	
  } catch (error) {
    console.error("Error uploading image:", error);
  }
}fetchImage();




onMount(async () => {
	  // Fetch data from the API
	  const response = await fetch('https://api.recruitly.io/api/business/profile?apiKey=TEST45684CB2A93F41FC40869DC739BD4D126D77');
	  const data = await response.json();
	  // Set the default phone number value from the API data
	  phone = data.phone;
	  console.log(phone);
  
	  const inputElement = document.querySelector('#phone-input');
	  iti = intlTelInput(inputElement, {
		utilsScript: 'https://cdn.jsdelivr.net/npm/intl-tel-input@16.0.3/build/js/utils.js',
		separateDialCode: true,
	  });
	  // Set the initial phone number value in the intlTelInput instance using E.164 format
	  iti.setNumber(phone);
  
	  // Set the placeholder number type to "FIXED_LINE"
	  iti.setPlaceholderNumberType('FIXED_LINE');
  
	  // Listen for the countrychange event
	  inputElement.addEventListener('countrychange', handleCountryChange);
  
	  // Listen for the input event to handle manual phone number changes
	  inputElement.addEventListener('input', handleInput);
	});
  
	function handleCountryChange() {
	  // Get the updated phone number with the new country code
	  phone = iti.getNumber(intlTelInputUtils.numberFormat.E164);
	}
  
	function handleInput() {
	  // Update the phone number with the country code
	  phone = iti.getNumber(intlTelInputUtils.numberFormat.INTERNATIONAL);
	}

  

onMount(async () => {


fetchCountries();

fetchCountryData ();
});




  </script>
    <main class="container">
	<form  class="form-container" method="POST">

		<div class="grid gap-6 mb-6 md:grid-cols-1">
			<div>
			  <Label>Company Profile</Label>
			  <hr>
			</div>
		  </div>

		  <div class="mb-6">
			<Label for="company_logo" class="mb-2">Company Logo</Label>
			<img id="uploadedImage" src="{imageUrl}" alt="Logo" on:click={handleLogoClick} style="cursor: pointer;" />
			
		</div>

		<Label>
			Company Name
			<Input name="name" autocomplete="off" bind:value={name}/>
		</Label>

		  
		<div class="mb-6">
			<Label for="address" class="mb-2">Address</Label>
			<Input type="text" id="address" bind:value={cityName} required />
	   </div>
		

	
			<div class="mb-6">
			<Label for="phone" class="mb-2" >Phone</Label>
			<Input type="tel" id="phone-input" autocomplete="off"  on:input={handleInput}  class="form-select  block w-full py-2.5 pl-3 pr-10 text-base border border-gray-300 rounded-lg bg-transparent focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm dark:text-white dark:focus:ring-gray-700 dark:focus:border-gray-600" style="width:500px;" />
	
		</div>

		<Label>
			Website
			<Input name="name" autocomplete="off" bind:value={website}/>
		</Label>

		<div class="mb-6">
			<Label for="currency" class="block text-sm font-medium text-gray-700 dark:text-white">Currency:</Label>
			<div class="relative mt-2 rounded-lg border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700">
			  <select id="currency" bind:value={currencyCode} required class="block w-full py-2.5 pl-3 pr-10 text-base border-transparent bg-transparent focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm dark:text-white dark:focus:ring-gray-700 dark:focus:border-gray-600">
				{#each currencyOptions as currency}
				<option value={currency.value}>{currency.label}</option>
				{/each}
			  </select>
			</div>
		
		  </div>
		<Label for="preferredDateFormat" class="block text-sm font-medium text-gray-700 dark:text-white">Preferred Date Format
		<div class="relative mt-2 rounded-lg border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700">
		  <Select id="preferredDateFormat" bind:value={preferredDateformat} autocomplete="off" required class="block w-full py-2.5 pl-3 pr-10 text-base border-transparent bg-transparent focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm dark:text-white dark:focus:ring-gray-700 dark:focus:border-gray-600">
			<option value="default" >Pretty (e.g., 1 Day Ago/2 Week Ago, etc.)</option>
			<option >Date only (e.g., 12/31/2020)</option>
			<option >Date and Time (e.g., 12/31/2020 15:00:00)</option>
			<option >Date and Time (e.g., 12/31/2020 03:00PM)</option>
		  </Select>
		</div>
	</Label>

	 
	<div class="mb-6">
		<Label for="timezone" class="block text-sm font-medium text-gray-700 dark:text-white">Timezone</Label>
		<div class="relative mt-2 rounded-lg border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700">
		  <select id="timezone" bind:value={timeZone} required class="block w-full py-2.5 pl-3 pr-10 text-base border-transparent bg-transparent focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm dark:text-white dark:focus:ring-gray-700 dark:focus:border-gray-600">
			{#each reactiveTimezoneOptions as timezone }
			<option value={timezone.code}>{timezone.name}</option>
			{/each}
		  </select>
		  </div>
	
	  </div>
	


		 

	  
	  <div class="mb-3">
		<Label for="country" class="mb-2">Default Country To Display</Label>
			<Select class="block w-full rounded-lg bg-white border border-gray-300 text-gray-700 focus:outline-none focus:border-indigo-500" bind:value={preferredCountryCode1} on:change={(event) => {
			   preferredCountryCode1 = event.target.value.toUpperCase();}}>
				  
					 {#each countryOptions as country1}
				   <option value={country1.value}>{country1.label}</option> 
				   {/each}
			</Select>
		 </div>


	
		 
	 <div class="mb-3">

		<Label class="mb-2">Preferred Countries To Display on Top</Label>
     {#if countries.length > 0}
      <MultiSelect items={countries} bind:value={selectedLabels} on:change={handleSelection} />
    {/if}

	</div>
	
		
		<div class="mb-6">
			<button type="button" on:click={updateData} class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
			  update profile
			</button>
		  </div>
	</form>
</main>

  
<style>
	.form-container {
	  margin-left: 550px;
	  width: 500px;
	 height: 200PX;
	  margin-top:-310px;
	}
	.container{
	 
	  width:600px;
	
	 
	}
	.mb-6 {
 
   border-radius: 1.25rem; 
}
	
  </style>