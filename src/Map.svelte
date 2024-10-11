<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import { feature } from 'topojson-client';
  
    let mapWrapper;
    let mapElement;
    let width;
    let height;
  
    const colorScale = (position) => {
      switch (position) {
        case 'republican': return '#B24846'; 
        case 'democrat': return '#0071E1'; // dark blue
        case 'battleground_trump': return '#DB908A'; // very light red
        case 'battleground_harris': return '#74B8FC'; // very light blue
        case 'lean_trump': return '#C86D6C'; // medium red
        case 'lean_harris': return '#0F7FEE'; // medium blue
        default: return '#ccc'; // default gray
      }
    };
  
    const normalizeStateName = (name) => {
      return name.toLowerCase().replace(/\s+/g, '_').replace(/\./g, '');
    };
  
    const stateAbbreviations = {
      "Alabama": "AL", "Alaska": "AK", "Arizona": "AZ", "Arkansas": "AR", "California": "CA",
      "Colorado": "CO", "Connecticut": "CT", "Delaware": "DE", "Florida": "FL", "Georgia": "GA",
      "Hawaii": "HI", "Idaho": "ID", "Illinois": "IL", "Indiana": "IN", "Iowa": "IA",
      "Kansas": "KS", "Kentucky": "KY", "Louisiana": "LA", "Maine": "ME", "Maryland": "MD",
      "Massachusetts": "MA", "Michigan": "MI", "Minnesota": "MN", "Mississippi": "MS", "Missouri": "MO",
      "Montana": "MT", "Nebraska": "NE", "Nevada": "NV", "New Hampshire": "NH", "New Jersey": "NJ",
      "New Mexico": "NM", "New York": "NY", "North Carolina": "NC", "North Dakota": "ND", "Ohio": "OH",
      "Oklahoma": "OK", "Oregon": "OR", "Pennsylvania": "PA", "Rhode Island": "RI", "South Carolina": "SC",
      "South Dakota": "SD", "Tennessee": "TN", "Texas": "TX", "Utah": "UT", "Vermont": "VT",
      "Virginia": "VA", "Washington": "WA", "West Virginia": "WV", "Wisconsin": "WI", "Wyoming": "WY",
      "District of Columbia": "DC"
    };

    // States to hide labels for
    const hiddenLabelStates = ['Vermont', 'New Hampshire', 'Massachusetts', 'Connecticut', 'Rhode Island', 'New Jersey', 'Delaware', 'District of Columbia'];
  
    $: if (mapWrapper) {
      width = mapWrapper.clientWidth;
      height = width; // Square aspect ratio
    }
  
    onMount(async () => {
      const resizeObserver = new ResizeObserver(() => {
        if (mapWrapper) {
          width = mapWrapper.clientWidth;
          height = width;
          updateMap();
        }
      });
  
      resizeObserver.observe(mapWrapper);
  
      await updateMap();
  
      return () => {
        resizeObserver.disconnect();
      };
    });
  
    async function updateMap() {
      if (!width || !height) return;
  
      const svg = d3.select(mapElement)
        .attr('width', width)
        .attr('height', height);
  
      svg.selectAll("*").remove(); // Clear previous content
  
      const projection = d3.geoAlbersUsa()
        .translate([width / 2, height / 2])
        .scale(width * 1.3);
  
      const path = d3.geoPath().projection(projection);
  
      try {
        const [us, csvData] = await Promise.all([
          d3.json('https://cdn.jsdelivr.net/npm/us-atlas@3/states-10m.json'),
          d3.csv('https://raw.githubusercontent.com/tomvaillant/us-wahl-data/refs/heads/main/map_widget.csv')
        ]);
  
        const states = feature(us, us.objects.states);
  
        // Create a map of normalized state names to positions
        const statePositions = new Map(csvData.map(d => [normalizeStateName(d.state), d.position]));
  
        const g = svg.append('g');
  
        g.selectAll('path')
          .data(states.features)
          .enter()
          .append('path')
          .attr('d', path)
          .attr('fill', d => {
            const stateName = normalizeStateName(d.properties.name);
            const position = statePositions.get(stateName);
            return colorScale(position);
          })
          .attr('stroke', '#fff')
          .attr('stroke-width', '0.5') // Reduced stroke width
          .append('title')
          .text(d => {
            const stateName = normalizeStateName(d.properties.name);
            return `${d.properties.name}: ${statePositions.get(stateName) || 'N/A'}`;
          });
  
        g.selectAll('text')
          .data(states.features)
          .enter()
          .append('text')
          .attr('transform', d => {
            const [x, y] = path.centroid(d);
            // Adjust position for Michigan
            if (d.properties.name === 'Michigan') {
              return `translate(${x + 3}, ${y + 5})`;
            }
            return `translate(${x}, ${y})`;
          })
          .attr('text-anchor', 'middle')
          .attr('dy', '.35em')
          .attr('font-size', '10px')
          .attr('font-weight', 'regular')
          .attr('fill', d => {
            const stateName = normalizeStateName(d.properties.name);
            const position = statePositions.get(stateName);
            return ['battleground_trump', 'battleground_harris', 'lean_trump', 'lean_harris'].includes(position) ? 'black' : 'white';
          })
          .text(d => {
            // Don't display labels for certain states
            if (hiddenLabelStates.includes(d.properties.name)) {
              return '';
            }
            return stateAbbreviations[d.properties.name] || '';
          });
  
      } catch (error) {
        console.error('Error loading or parsing data:', error);
      }
    }
  </script>
  
  <div bind:this={mapWrapper} class="map-wrapper">
    <svg class="us-elections-map" bind:this={mapElement}></svg>
  </div>
  
  <style>
    .map-wrapper {
      width: 100%;
    }
  
    svg {
      display: block;
      width: 100%;
      margin: 0;
    }
  </style>