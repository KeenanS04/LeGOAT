<script>
  import { onMount } from 'svelte';
  import { fade } from 'svelte/transition';
  import Scroller from "@sveltejs/svelte-scroller";
  import ScoringLeadersChart from './ScoringLeadersChart.svelte';
  import MapComp from './map.svelte';
  import MapOvr from './map_ovr.svelte';
  import * as d3 from 'd3';

  let count, index, offset, progress;
  index = 0;
  let width, height;


  let nbaData;
  let selectedYear = 2004;
  let chartData = [];

  let visualizations_hidden = true;
  $: console.log(index);
  $: {
    if (index === 0 || index > sections.length) {
      visualizations_hidden = true;
    } else if (offset > .6) {
      visualizations_hidden = false;
    }
  }

let sections = [
    {
      title: 'Origins: Growing up in Akron',
      content: `LeBron Raymone James was born on December 30, 1984, in Akron, Ohio. His mother, Gloria James, raised him as a single parent. LeBron's childhood was marked by poverty and instability, but his natural talent and work ethic set him apart. He quickly rose through the ranks of youth basketball, earning national attention and acclaim.`,
      imageUrl: 'images/chosen_one.ong.webp',
      imageClass: 'origins-image'
    },
    {
      title: 'Prodigy to Phenom: The Foundation in Cleveland (2003 - 2010)',
      content: `LeBron James, the kid from Akron, Ohio, burst onto the professional basketball scene with the promise of greatness. His impact was immediate, earning the NBA Rookie of the Year award and an All-Star Game appearance in his debut season. His rare combination of size, skill, and intelligence reinvigorated the Cleveland Cavaliers. By his fourth season, he led the Cavs to their first NBA Finals appearance in franchise history. Though they fell short, LeBron had already begun etching his name into the annals of basketball lore.

      In Cleveland, LeBron's individual accolades piled up swiftly. He was a six-time All-Star, a two-time All-Star Game MVP, and secured two regular-season MVPs. He led the league in scoring in 2008 and was perennially named to the All-NBA First Team and the NBA All-Defensive First Team, showcasing his two-way prowess. However, the championship glory that Cleveland so desperately craved remained just out of reach.`,
      imageUrl: 'images/youngbron.webp',
      imageClass: 'draft-image'
    },
    // {
    //   title: 'He is Special',
    //   content: `LeBron's NBA debut against the Sacramento Kings is memorable, as he scores 25 points, announcing his arrival with a performance that earns him the Rookie of the Year award.
    //   LeBron leads the Cavaliers to the 2007 NBA Finals, their first Finals appearance in franchise history, only to be swept by the San Antonio Spurs. This series sets the stage for LeBron's quest for an NBA title and marks the beginning of his challenging rivalry with the Spurs.
    //   After a disappointing bronze in 2004, LeBron, as part of the 'Redeem Team,' secures a gold medal in the 2008 Olympics, symbolizing a resurgence in American basketball dominance.`,
    //   imageUrl: '/path/to/image1.jpg'
    // },
    {
      title: 'The Heat is ON: The Decision, Champions and Challenges in Miami (2010 - 2014)',
      content: `The Decision to join the Miami Heat changed the NBA's balance of power and LeBron's career trajectory. In Miami, LeBron refined his game, becoming more efficient and deadly, particularly from beyond the arc. He garnered two more MVP awards and led the Heat to four straight NBA Finals, winning two. His playoff performances during this period were nothing short of legendary, including a 45-point effort against the Boston Celtics to stave off elimination in 2012 and a 37-point closing masterpiece in Game 7 of the 2013 Finals.

      Off the court, LeBron's influence grew. He was not only a fixture at All-Star weekends but also a global ambassador for the sport, representing Team USA. He captured two Olympic medals during this period – gold in Beijing (2008) and London (2012), further cementing his status as basketball royalty.`,
      imageUrl: 'images/miami.webp',
      imageClass: 'heat-image'
    },
    // {
    //   title: 'Dominance in Miami',
    //   content: `LeBron's first season with the Heat culminates in a Finals appearance against the Dallas Mavericks. However, the Heat lose in six games, a series that serves as a crucial learning experience for LeBron and motivates his evolution as a player. LeBron wins the NBA MVP award in 2012 and 2013, solidifying his status as the league's top player during the Heat's championship era. LeBron leads Miami to NBA championships in 2012 and 2013, overcoming the Oklahoma City Thunder and the San Antonio Spurs. These victories mark the fulfillment of his championship aspirations and contribute to his growing legacy. Additionally, In March 2014, against the Charlotte Bobcats, LeBron scored a career-high 61 points. This unforgettable performance showcased his scoring versatility, marking a high point in his Miami Heat tenure. Securing another gold medal with Team USA in 2012, LeBron further cemented his legacy as a basketball icon. His contributions were key in maintaining Team USA's dominance in international basketball.
    //   `,
    //   imageUrl: '/path/to/image1.jpg'
    // },
    {
      title: 'The Prodigal Son Returns: Triumph and Turmoil back in Cleveland (2014 - 2018)',
      content: `LeBron's return to Cleveland was about redemption and delivering on a long-held promise. He led the Cavaliers to four consecutive NBA Finals, including the miraculous 2016 championship win that not only shattered Cleveland's curse but also solidified his clutch legacy. In these years, he continued to stack up All-NBA First Team selections and All-Star appearances, consistently dazzling fans with his high basketball IQ and versatility.

      LeBron's 2018 playoff run was one for the ages, often single-handedly carrying the Cavs through each round, including a buzzer-beater against the Indiana Pacers that added to his highlight reel of iconic moments.`,
      imageUrl: 'images/cavs.webp',
      imageClass: 'cavaliers-image'
    },
    // {
    //   title: 'This one is for you Cleveland',
    //   content: `LeBron leads the Cavaliers to a historic comeback from a 3-1 deficit against the 73-win Golden State Warriors in the 2016 NBA Finals. His chase-down block in Game 7 becomes an iconic moment, and the victory ends Cleveland's 52-year championship drought across all major sports. Throughout his second stint with Cleveland, LeBron's Cavaliers consistently overcome Eastern Conference rivals, including memorable performances against the Toronto Raptors, earning the nickname 'LeBronto' for his domination of the team in the playoffs.`,
    //   imageUrl: '/path/to/image1.jpg'
    // },
    {
      title: 'The L.A story: Cementing a Legacy (2018 - Present)',
      content: `With the Lakers, LeBron's narrative took on a new dimension. His leadership and experience helped capture the 2020 NBA Championship, a poignant triumph in the wake of Kobe Bryant's tragic passing. He continued to break records, becoming the oldest player to average a triple-double over an entire month. LeBron also earned his 17th All-Star selection in 2021, tying with Kobe Bryant and trailing only Kareem Abdul-Jabbar and Karl Malone. The lakers also won the inaugral In Season Tournament with LeBron finishing with tourney MVP.
      
      In 2022, at the age of 37, LeBron achieved a historic milestone by becoming the all-time leading scorer in NBA Playoffs history, surpassing Michael Jordan. On February 7, 2023, LeBron James made history by surpassing Kareem Abdul-Jabbar to become the NBA's all-time leading scorer, using his signature fadeaway jumper to seal the record. As of February 29, 2024, he is just 40 points away from reaching the unprecedented milestone of 40,000 career points, poised to set yet another record in his illustrious career.`,
      imageUrl: 'images/lakerBron.webp',
      imageClass: 'la-image'
    },
    // {
    //   title: 'Legacy',
    //   content: `Off the hardwood, LeBron's impact has been felt in education through his "I PROMISE School," philanthropy, and activism, particularly in his staunch advocacy for racial justice and voter rights. His media company, SpringHill Entertainment, produced the movie "Space Jam: A New Legacy," where he starred as the lead, blending his athletic and artistic pursuits.`,
    //   imageUrl: '/path/to/image1.jpg'
    // },
  ];

  async function loadData() {
    await d3.csv('nba_data.csv').then(data => {
      nbaData = data;
      updateChartData(selectedYear);
    });
  }

  $: if (nbaData) {
    let newYear = selectedYear + 5 * (index-1);
    updateChartData(newYear <= 2023 ? newYear : 2023);
  }
  // $: console.log(chartData);

  function updateChartData(year) {
    const scores = nbaData.map(d => ({
      player: d['Leaders Names'],
      score: +d[year]
      //imageURL: d['Image URL'] // Assuming this is where player images are stored
    })); 

    scores.sort((a, b) => b.score - a.score);

    let topScorers = scores.slice(0, 15);

    if (!topScorers.some(d => d.player.includes('#23 LeBron James'))) {
      const leBron = scores.find(d => d.player.includes('#23 LeBron James'));
      if (leBron) topScorers.push(leBron);
    }

    chartData = topScorers;
  }

  // let currentSection = sections[0]; // Initialize with the first section to have a default
  // let currentIndex = 0;

  // $: currentIndex = Math.floor(index / 2),
  // currentSection = sections[currentIndex] || sections[0];
  // $: console.log("Current index:", index);

  onMount(() => {
    loadData();
  });

  $: fadeInClass = index > sections.length-1 ? 'fadeInAnimation' : '';
  $: slideInLeftClass = index > sections.length-1 ? 'slideInLeftAnimation' : '';
  $: slideInBottomClass = index > sections.length-1 ? 'slideInBottomAnimation' : '';

</script>

<Scroller
  top={0.0}
  bottom={1}
  threshold={1}
  bind:count
  bind:index
  bind:offset
  bind:progress
>
  <div class="background" slot="background" bind:clientWidth={width} bind:clientHeight={height}>
    <div style="transition: opacity 0.5s; opacity: {visualizations_hidden ? 0 : 1}">
        <MapComp {index}/>
        <ScoringLeadersChart {chartData} {index}/>
    </div>
  </div>
    
  <div class="foreground" slot="foreground">
    <section class="hero">
      <h1>LeGOAT: The LeBron James's Career Narrative</h1>
      <h2 class='subhead'>Is LeBron James's Unparalleled Scoring Prowess the Defining Factor of His Claim to the Basketball Throne?</h2>
      <img src='images/lob.webp' alt='LeBron James' class="header-image"/>
      <iframe width="420" height="315"
              src="https://www.youtube.com/watch?v=wze5XLenrSc"
              title="LeBron James Highlight Reel"
              class="header-image">
      </iframe>
    </section>
    {#each sections as section, i}
      <section class="scrollable-section">
        {#if i+1 === index}
          <div class="content" in:fade={{duration: 400 }} out:fade={{duration: 500 }}>
            <h1>{section.title}</h1>
            <p class='left'>{section.content}</p>
            <div>
              {#if section.imageUrl}
                <img src={section.imageUrl} alt={section.title} class="section-image {section.imageClass}"/>
              {/if}
            </div>
          </div>
        {/if}
      </section>
    {/each}
    <section class="container">
      <div class="map-accolades">
          <div class="map-container" {fadeInClass}>
              <h1>LeBron's Jouney And Accolades Throughout His Career</h1>
              <p> Click on each logo to check out LeBron's accolades on each team</p>
              <MapOvr/>
          </div>
          <div class="conclusion" {slideInLeftClass}>
              <h2>Conclusion</h2>
              <p>LeBron came into the NBA with great expectations, and for the first part of his career he performed up to par with some of the greats in the game. But to solidify his legacy as a true legend of the sport, he needed to start winning more. Therefore he switched teams and worked together with other stars to win his first two championships with the Miami Heat. After completing everything in the books, LeBron wanted a name for himself and his city, and in his path stood the Warriors dynasty. Against all odds, he still ended up overcoming the Warriors to win a championship for his home team. In recent times, LeBron has done philanthropic work with his I Promise School and focused more on his family in LA. LeBron is the definition of growth, hard work, talent, and perseverance, and his journey proves just why he is the basketball GOAT.</p>
          </div>
      </div>
  
      <aside class="accolades-list" {slideInBottomClass}>
          <h3 style="text-align: center;">LeBron's NBA Accolades</h3>
          <ul>
            <li>4× NBA champion (2012, 2013, 2016, 2020)</li>
            <li>4× NBA Finals MVP (2012, 2013, 2016, 2020)</li>
            <li>4× NBA Most Valuable Player (2009, 2010, 2012, 2013)</li>
            <li>20× NBA All-Star (2005–2024)</li>
            <li>3× NBA All-Star Game MVP (2006, 2008, 2018)</li>
            <li>13× All-NBA First Team (2006, 2008–2018, 2020)</li>
            <li>3× All-NBA Second Team (2005, 2007, 2021)</li>
            <li>3× All-NBA Third Team (2019, 2022, 2023)</li>
            <li>5× NBA All-Defensive First Team (2009–2013)</li>
            <li>NBA All-Defensive Second Team (2014)</li>
            <li>NBA Rookie of the Year (2004)</li>
            <li>NBA All-Rookie First Team (2004)</li>
            <li>NBA scoring champion (2008)</li>
            <li>NBA assists leader (2020)</li>
            <li>NBA 75th Anniversary Team</li>
            <li>AP Athlete of the Decade (2010s)</li>
            <li>4× AP Athlete of the Year (2013, 2016, 2018, 2020)</li>
            <li>3× Sports Illustrated Sportsperson of the Year (2012, 2016, 2020)</li>
            <li>USA Basketball Male Athlete of the Year (2012)</li>
            <li>NBA In Season Tournament Champion (2024)</li>
            <li>NBA In Season Tournament MVP (2024)</li>
        </ul>
      </aside>
    </section>
  </div>
</Scroller>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&display=swap');
  @import 'https://fonts.googleapis.com/css?family=Shrikhand|Yantramanav';
  @import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;700&display=swap');

  @keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
  }
  @keyframes slideInLeft {
    from { transform: translateX(-100%); }
    to { transform: translateX(0); }
  }
  @keyframes slideInBottom {
    from { transform: translateY(100%); }
    to { transform: translateY(0); }
  }

  @keyframes slideInFromLeft {
    from { transform: translateX(-100%); opacity: 0; }
    to { transform: translateX(0); opacity: 1; }
  }

  @keyframes slideInFromTop {
    from { transform: translateY(-100%); opacity: 0; }
    to { transform: translateY(0); opacity: 1; }
  }

  /* .slideInFromLeft {
    animation: slideInFromLeft 1s ease-out forwards;
  }

  .slideInFromTop {
    animation: slideInFromTop 1s ease-out forwards;
  } */

  :global(body) {
    background: linear-gradient(to right, #b0b9ba, #e6ebee, #b0b9ba);
  }

  .subhead {
    animation: slideInFromTop 0.5s ease-out forwards;
  }
  section {
    font-family: 'Yantramanav', sans-serif;
    height: 100vh;
    text-align: center;
    max-width: 100%; 
    color: black;
    padding: 2em;
    margin: 0 0 0 0;
    border-radius: 6px;
  }

  .scrollable-section {
    position: sticky;
    top: 0;
  }

  .foreground {
    width: 100%;
    margin: 0 auto;
    height: auto;
    position: relative;
  }

  .hero h1 {
    font-size: 3em;
  }

  .hero img {
    max-width: 100%; 
    max-height: 60vh; 
    object-fit: contain;
  }

  .content {
    height: 93vh;
    width: 45vw;
    /* border: black 1px solid; */
    position: absolute;
    left: 53%;
    border: black 2px solid;
    border-radius: 5px;
    background-color: rgba(255, 255, 255, 0.5);
    /* background-color: #fbf6ed; */
  }

  .content h1 {
    font-size: 2em;
    margin-top: 20px;
  }

  /* .content::before {
    content: '';
    position: absolute;
    right: 101.5%;
    width: 2px; 
    background-color: black; 
    height: 99%;
  }
  .content::after {
    content: '';
    position: absolute;
    left: -116%;
    right: 103%;
    top: 52.5%;
    height: 2px;
    background-color: black;
  } */

  .content h1 {
    font-size: 2em;
    margin-top: 20px;
  }

  .left{
    text-align: center;
    font-size: 20px;
    margin-top: 20px;
    padding-left: 10px;
    padding-right: 10px;
  }

  .header-image{
    animation: slideInBottom 0.5s ease-out forwards;
    height: 40%;
    margin: 10px;
  }

  .cavaliers-image{
    max-width: 80%; 
    max-height: 40vh; 
    object-fit: contain;
    margin-top: 10px;
  }
  .draft-image{
    max-width: 80%; 
    max-height: 30vh; 
    object-fit: contain;
    margin-top: 10px;
  }
  .heat-image{
    max-width: 80%; 
    max-height: 40vh; 
    object-fit: contain;
  }
  .cavaliers-image{
    max-width: 80%; 
    max-height: 45vh; 
    object-fit: contain;
  }
  .la-image{
    max-width: 80%; 
    max-height: 30vh; 
    object-fit: contain;
  }
  .origins-image{
    max-width: 80%; 
    max-height: 60vh; 
    object-fit: contain;
    margin-top: 10px;
  }
  .container {
    display: grid;
    grid-template-columns: 3fr 1fr; 
    grid-template-rows: auto 1fr; /* Auto for the header, 1fr for the content */
    gap: 20px; /* Space between grid items */
    height: 95vh; /* Full viewport height */
  }

  .map-accolades {
      display: flex;
      flex-direction: column; /* Stack children vertically */
  }

  .map-container {
      animation: fadeIn 0.5s ease-out forwards;
  }

  .map-container, .conclusion {
      flex-grow: 1; /* Allow these elements to fill available space */
  }

  .accolades-list {
      animation: slideInLeft 0.5s ease-out forwards;
      background-color: rgba(255, 255, 255, 0.5);
      border-radius: 8px; /* Optional: adds rounded corners */
      height: 95vh;
      padding-left: 20px;
      padding-right: 20px;
      text-align: left;
  }
  /* .accolades-list::before {
    content: '';
    position: absolute;
    right: 102%;
    width: 2px; 
    background-color: black; 
    height: 99%;
  } */

  .conclusion {
      animation: slideInBottom 0.5s ease-out forwards;
  }
  li{
    padding: 0.5px
  }
  h1 {
    font-family: 'Bebas Neue', sans-serif;
    /* font-family: 'Playfair Display', serif; */
    font-size: 3em; 
    font-weight: 700;
    letter-spacing: 2px;
  }

  

  /* h1, h2, h3 {
    font-family: 'Playfair Display', serif;
    color: #333; 
    margin: 0.5em 0; 
  }

  h1 {
    font-size: 2.5em; 
    font-weight: 700; 
  }

  h2 {
    font-size: 2em; 
    font-weight: 700;
  }

  h3 {
    font-size: 1.75em;
    font-weight: 700; 
  }

  p {
    font-family: 'Playfair Display', serif;
    font-size: 12px; 
    color: #333; 
    margin: 0 0 1em 0; 
  } */

</style>
