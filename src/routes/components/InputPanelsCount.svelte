<!--
 Copyright 2023 Google LLC

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

      https://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
 -->

<script lang="ts">
  import type { MdSlider } from '@material/web/slider/slider';
  import type { SolarPanelConfig } from '../solar';
  import { onMount } from 'svelte';

  export let configId: number;
  export let solarPanelConfigs: SolarPanelConfig[];

  function onChange(event: Event) {
    const target = event.target as MdSlider;
    configId = target.value ?? 0;
  }
  onMount(() => {
    // Function to get a query parameter by name
    function getQueryParam(param: string): string | null {
      const urlParams = new URLSearchParams(window.location.search);
      return urlParams.get(param);
    }

    const panelCountParam = getQueryParam('panel-count');
    console.log('Panel count parameter from URL:', panelCountParam);

    let panelCountFromUrl = 4; // default value
    if (panelCountParam !== null) {
      const parsedPanelCount = parseInt(panelCountParam, 10);
      if (!isNaN(parsedPanelCount)) {
        panelCountFromUrl = parsedPanelCount;
      } else {
        console.warn('Invalid panel count value in URL. Using default value of 4.');
      }
    } else {
      console.warn('Panel count parameter not found in URL. Using default value of 4.');
    }
    console.log('Panel count to be used:', panelCountFromUrl);

    // Find the configId based on the panel count
    const configIndex = solarPanelConfigs.findIndex(
      (config) => config.panelsCount === panelCountFromUrl,
    );
    console.log('Config index found:', configIndex);

    if (configIndex !== -1) {
      configId = configIndex;
    } else {
      console.warn('Panel count not found in configurations. Setting to default configId.');
      configId = solarPanelConfigs.findIndex((config) => config.panelsCount === 4);
    }
  });
</script>

<div>
  <table class="table-auto w-full body-medium secondary-text">
    <tr>
      <td class="primary-text"><md-icon>solar_power</md-icon> </td>
      <th class="pl-2 text-left">Panels count</th>
      <td class="pl-2 text-right">
        <span>{solarPanelConfigs[configId].panelsCount} panels</span>
      </td>
    </tr>
  </table>

  <md-slider
    class="w-full"
    value={configId}
    min={0}
    max={solarPanelConfigs.length - 1}
    on:change={onChange}
  />
</div>
