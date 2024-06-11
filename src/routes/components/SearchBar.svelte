<!-- <script lang="ts">
  /* global google */

  import { onMount } from 'svelte';
  import type { MdFilledTextField } from '@material/web/textfield/filled-text-field';

  export let location: google.maps.LatLng | undefined;

  export let placesLibrary: google.maps.PlacesLibrary;
  export let map: google.maps.Map;
  export let initialValue = '';
  export let zoom = 19;

  let textFieldElement: MdFilledTextField;
  function getQueryParam(name: string): string | null {
    const urlParams = new URLSearchParams(window.location.search);
    return urlParams.get(name);
  }
  onMount(async () => {
    // Wait for the text field to be rendered
    await textFieldElement.updateComplete;
    const inputElement = textFieldElement.renderRoot.querySelector('input') as HTMLInputElement;
    const autocomplete = new placesLibrary.Autocomplete(inputElement, {
      fields: ['formatted_address', 'geometry', 'name', 'address_components'],
    });

    autocomplete.addListener('place_changed', async () => {
      const place = autocomplete.getPlace();
      if (!place.geometry || !place.geometry.location) {
        textFieldElement.value = '';
        return;
      }
      if (place.geometry.viewport) {
        map.setCenter(place.geometry.location);
        map.setZoom(zoom);
      } else {
        map.setCenter(place.geometry.location);
        map.setZoom(zoom);
      }

      location = place.geometry.location;
      if (place.name) {
        textFieldElement.value = place.name;
      } else if (place.formatted_address) {
        textFieldElement.value = place.formatted_address;
      }

      // Extracting state from the place data
      let state = '';
      if (place.address_components) {
        for (let i = 0; i < place.address_components.length; i++) {
          const component = place.address_components[i];
          console.log('Component type:', component.types);
          if (component.types.includes('administrative_area_level_1')) {
            state = component.short_name;
            break; // No need to continue once state is found
          }
        }
      }

      // Now 'state' variable holds the state information
      console.log('State:', state);
    });

    // Check if there is an address in the URL
    const addressFromUrl = getQueryParam('address');
    if (addressFromUrl) {
      inputElement.value = addressFromUrl;
      const placesService = new google.maps.places.PlacesService(map);
      placesService.textSearch({ query: addressFromUrl }, (results, status) => {
        if (status === google.maps.places.PlacesServiceStatus.OK && results && results[0]) {
          const place = results[0];
          if (place.geometry && place.geometry.location) {
            map.setCenter(place.geometry.location);
            map.setZoom(zoom);
            location = place.geometry.location;
            if (place.name) {
              textFieldElement.value = place.name;
            } else if (place.formatted_address) {
              textFieldElement.value = place.formatted_address;
            }
          }
        }
      });
    }
  });
</script>

<md-filled-text-field bind:this={textFieldElement} label="Search an address" value={initialValue}>
  <md-icon slot="leadingicon">search</md-icon>
</md-filled-text-field> -->
<script lang="ts">
  /* global google */

  import { onMount } from 'svelte';
  import type { MdFilledTextField } from '@material/web/textfield/filled-text-field';
  import { createEventDispatcher } from 'svelte';
  import SolarPotentialSection from '../sections/SolarPotentialSection.svelte';
  export let location: google.maps.LatLng | undefined;

  export let placesLibrary: google.maps.PlacesLibrary;
  export let map: google.maps.Map;
  export let initialValue = '';
  export let zoom = 19;

  let textFieldElement: MdFilledTextField;
  let selectedStateRate = 0.31;

  async function fetchData() {
    try {
      const response = await fetch('./src/zipcode.json');

      if (!response.ok) {
        throw new Error('Failed to fetch data');
      }
      const data = await response.json();
      console.log(data, 'datatdadtda');
      return data;
    } catch (error) {
      console.error('Error fetching data:', error);
      return null;
    }
  }

  function getQueryParam(name: string): string | null {
    const urlParams = new URLSearchParams(window.location.search);
    return urlParams.get(name);
  }

  onMount(async () => {
    const energyData = await fetchData();

    // Wait for the text field to be rendered
    await textFieldElement.updateComplete;
    const inputElement = textFieldElement.renderRoot.querySelector('input') as HTMLInputElement;
    const autocomplete = new placesLibrary.Autocomplete(inputElement, {
      fields: ['formatted_address', 'geometry', 'name', 'address_components'],
    });

    autocomplete.addListener('place_changed', async () => {
      const place = autocomplete.getPlace();
      if (!place.geometry || !place.geometry.location) {
        textFieldElement.value = '';
        return;
      }
      if (place.geometry.viewport) {
        map.setCenter(place.geometry.location);
        map.setZoom(zoom);
      } else {
        map.setCenter(place.geometry.location);
        map.setZoom(zoom);
      }

      location = place.geometry.location;
      if (place.name) {
        textFieldElement.value = place.name;
      } else if (place.formatted_address) {
        textFieldElement.value = place.formatted_address;
      }

      // Extracting state from the place data
      let state = '';
      if (place.address_components) {
        for (let i = 0; i < place.address_components.length; i++) {
          const component = place.address_components[i];
          if (component.types.includes('administrative_area_level_1')) {
            state = component.short_name;
            break; // No need to continue once state is found
          }
        }
      }

      // Now 'state' variable holds the state information
      console.log('State:', state);

      // Find the corresponding average residential rate based on the selected state
      if (energyData) {
        const selectedEnergyData = energyData.find(
          (data: { state: string }) => data.state === state,
        );
        if (selectedEnergyData) {
          selectedStateRate = selectedEnergyData.avg_residential_rate;
          localStorage.setItem('selectedStateRate', JSON.stringify(selectedStateRate));
          console.log('Selected State Rate:', selectedStateRate);
          // Now 'selectedStateRate' holds the average residential rate for the selected state
          // You can pass this to another component or use it as needed
        } else {
          selectedStateRate = 0.31;
          localStorage.setItem('selectedStateRate', JSON.stringify(selectedStateRate));
          console.log('State data not found for:', state);
        }
      } else {
        console.error('Energy data not available');
      }
    });

    // Check if there is an address in the URL
    const addressFromUrl = getQueryParam('address');
    if (addressFromUrl) {
      inputElement.value = addressFromUrl;
      const placesService = new google.maps.places.PlacesService(map);
      placesService.textSearch({ query: addressFromUrl }, (results, status) => {
        if (status === google.maps.places.PlacesServiceStatus.OK && results && results[0]) {
          const place = results[0];
          if (place.geometry && place.geometry.location) {
            map.setCenter(place.geometry.location);
            map.setZoom(zoom);
            location = place.geometry.location;
            if (place.name) {
              textFieldElement.value = place.name;
            } else if (place.formatted_address) {
              textFieldElement.value = place.formatted_address;
            }

            // Extract the state from the place data
            let state = '';
            if (place.address_components) {
              for (let i = 0; i < place.address_components.length; i++) {
                const component = place.address_components[i];
                if (component.types.includes('administrative_area_level_1')) {
                  state = component.short_name;
                  break; // No need to continue once state is found
                }
              }
            }

            // Find the corresponding average residential rate based on the selected state
            if (energyData) {
              const selectedEnergyData = energyData.find(
                (data: { state: string }) => data.state === state,
              );
              if (selectedEnergyData) {
                selectedStateRate = selectedEnergyData.avg_residential_rate;
                localStorage.setItem('selectedStateRate', JSON.stringify(selectedStateRate));
                console.log('Selected State Rate:', selectedStateRate);
                // Now 'selectedStateRate' holds the average residential rate for the selected state
                // You can pass this to another component or use it as needed
              } else {
                selectedStateRate = 0.31;
                localStorage.setItem('selectedStateRate', JSON.stringify(selectedStateRate));
                console.log('State data not found for:', state);
              }
            } else {
              console.error('Energy data not available');
            }
          }
        }
      });
    }
  });
</script>

<md-filled-text-field bind:this={textFieldElement} label="Search an address" value={initialValue}>
  <md-icon slot="leadingicon">search</md-icon>
</md-filled-text-field>
