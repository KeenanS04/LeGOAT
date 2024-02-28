<script>
    export let year;
    export let isActive = false;
  </script>
  
  <div class="section" class:active={isActive}>
    <h2>Year: {year}</h2>
    <p>LeBron James's highlights of {year}</p>
    <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit...</p>
  </div>
  
  <style>
    .section {
      background-color: #f5f5dc; /* A light tan background, resembling paper */
      border: 3px solid #800000; /* Maroon/dark red border */
      padding: 20px;
      margin: 10px 0;
      transition: border-color 0.3s ease-in-out;
    }
  
    .section h2 {
      color: #333;
      font-family: 'Times New Roman', serif;
    }
  
    .section.active {
      border-color: #a52a2a; /* A slightly lighter shade of maroon for the active section */
    }
  
    /* Add styles for the p tag if needed */
  </style>