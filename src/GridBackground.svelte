<script>
    export let size = 40; 
    export let color = "rgba(188, 150, 255, 0.2)";
    export let plusSize = 12;
  </script>
  
  <div class="grid-background">
    <svg width="100%" height="100%">
      <defs>
        <pattern id="plus-pattern" width="{size}" height="{size}" patternUnits="userSpaceOnUse">
          <path d="M {size/2 - plusSize/2} {size/2} h{plusSize} M {size/2} {size/2 - plusSize/2} v{plusSize}" 
                stroke={color} 
                stroke-width="1" 
                stroke-linecap="round"/>
        </pattern>
      </defs>
      <rect width="100%" height="100%" fill="url(#plus-pattern)" />
    </svg>
  </div>
  
  <style>
    .grid-background {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      z-index: -1;
      pointer-events: none;
    }
    
    svg {
      display: block;
    }
  </style>