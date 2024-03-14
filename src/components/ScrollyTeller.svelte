<script>
  import { onMount } from 'svelte';
  import { fade } from 'svelte/transition';
  import Scroller from "@sveltejs/svelte-scroller";
  import ScoringLeadersChart from './ScoringLeadersChart.svelte';
  import MapComp from './map.svelte';
  import * as d3 from 'd3';

  let count, index, offset, progress;
  let width, height;


  let nbaData;
  let selectedYear = 2004;
  let chartData = [];

let sections = [
    {
      title: 'LeGOAT: The LeBron James Story',
      content: '',
      imageUrl: 'src/images/lob.webp'
    },
    {
      title: 'Origins: Growing up in Akron',
      content: `LeBron Raymone James was born on December 30, 1984, in Akron, Ohio. His mother, Gloria James, raised him as a single parent. LeBron's childhood was marked by poverty and instability, but his natural talent and work ethic set him apart. He quickly rose through the ranks of youth basketball, earning national attention and acclaim.`,
      imageUrl: 'src/images/hsBron.jpeg'
    },
    {
      title: 'Prodigy to Phenom: The Foundation in Cleveland (2003 - 2010)',
      content: `LeBron James, the kid from Akron, Ohio, burst onto the professional basketball scene with the promise of greatness. His impact was immediate, earning the NBA Rookie of the Year award and an All-Star Game appearance in his debut season. His rare combination of size, skill, and intelligence reinvigorated the Cleveland Cavaliers. By his fourth season, he led the Cavs to their first NBA Finals appearance in franchise history. Though they fell short, LeBron had already begun etching his name into the annals of basketball lore.

      In Cleveland, LeBron's individual accolades piled up swiftly. He was a six-time All-Star, a two-time All-Star Game MVP, and secured two regular-season MVPs. He led the league in scoring in 2008 and was perennially named to the All-NBA First Team and the NBA All-Defensive First Team, showcasing his two-way prowess. However, the championship glory that Cleveland so desperately craved remained just out of reach.`,
      imageUrl: 'src/images/youngbron.jpg'
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
      imageUrl: 'src/images/miami.png'
    },
    // {
    //   title: 'Dominance in Miami',
    //   content: `LeBron's first season with the Heat culminates in a Finals appearance against the Dallas Mavericks. However, the Heat lose in six games, a series that serves as a crucial learning experience for LeBron and motivates his evolution as a player. LeBron wins the NBA MVP award in 2012 and 2013, solidifying his status as the league's top player during the Heat's championship era. LeBron leads Miami to NBA championships in 2012 and 2013, overcoming the Oklahoma City Thunder and the San Antonio Spurs. These victories mark the fulfillment of his championship aspirations and contribute to his growing legacy. Additionally, In March 2014, against the Charlotte Bobcats, LeBron scored a career-high 61 points. This unforgettable performance showcased his scoring versatility, marking a high point in his Miami Heat tenure. Securing another gold medal with Team USA in 2012, LeBron further cemented his legacy as a basketball icon. His contributions were key in maintaining Team USA's dominance in international basketball.,
    //   `
    //   imageUrl: '/path/to/image1.jpg'
    // },
    {
      title: 'The Prodigal Son Returns: Triumph and Turmoil back in Cleveland (2014 - 2018)',
      content: `LeBron's return to Cleveland was about redemption and delivering on a long-held promise. He led the Cavaliers to four consecutive NBA Finals, including the miraculous 2016 championship win that not only shattered Cleveland's curse but also solidified his clutch legacy. In these years, he continued to stack up All-NBA First Team selections and All-Star appearances, consistently dazzling fans with his high basketball IQ and versatility.

      LeBron's 2018 playoff run was one for the ages, often single-handedly carrying the Cavs through each round, including a buzzer-beater against the Indiana Pacers that added to his highlight reel of iconic moments.`,
      imageUrl: 'src/images/cavs.webp'
    },
    // {
    //   title: 'This one is for you Cleveland',
    //   content: `LeBron leads the Cavaliers to a historic comeback from a 3-1 deficit against the 73-win Golden State Warriors in the 2016 NBA Finals. His chase-down block in Game 7 becomes an iconic moment, and the victory ends Cleveland's 52-year championship drought across all major sports. Throughout his second stint with Cleveland, LeBron's Cavaliers consistently overcome Eastern Conference rivals, including memorable performances against the Toronto Raptors, earning the nickname 'LeBronto' for his domination of the team in the playoffs.`,
    //   imageUrl: '/path/to/image1.jpg'
    // },
    {
      title: 'The L.A story: Cementing a Legacy (2018 - Present)',
      content: `With the Lakers, LeBron's narrative took on a new dimension. His leadership and experience helped capture the 2020 NBA Championship, a poignant triumph in the wake of Kobe Bryant's tragic passing. He continued to break records, becoming the oldest player to average a triple-double over an entire month. LeBron also earned his 17th All-Star selection in 2021, tying with Kobe Bryant and trailing only Kareem Abdul-Jabbar and Karl Malone.
      
      In 2022, at the age of 37, LeBron achieved a historic milestone by becoming the all-time leading scorer in NBA Playoffs history, surpassing Michael Jordan. On February 7, 2023, LeBron James made history by surpassing Kareem Abdul-Jabbar to become the NBA's all-time leading scorer, using his signature fadeaway jumper to seal the record. As of February 29, 2024, he is just 40 points away from reaching the unprecedented milestone of 40,000 career points, poised to set yet another record in his illustrious career.
      
      Off the hardwood, LeBron's impact has been felt in education through his "I PROMISE School," philanthropy, and activism, particularly in his staunch advocacy for racial justice and voter rights. His media company, SpringHill Entertainment, produced the movie "Space Jam: A New Legacy," where he starred as the lead, blending his athletic and artistic pursuits.`,
      imageUrl: 'src/images/lakerBron.jpg'
    },
  ];

  async function loadData() {
    await d3.csv('nba_data.csv').then(data => {
      nbaData = data;
      updateChartData(selectedYear);
    });
  }

  $: if (nbaData) {
    let newYear = selectedYear + 5 * (Math.floor(index/2)-1);
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

    let topScorers = scores.slice(0, 10);

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
    <div>
      <MapComp {index}/>
    </div>
    <div class="chart-container" in:fade={{ duration: 400 }} out:fade={{ duration: 400 }}>
      <ScoringLeadersChart {chartData} {index}/>
    </div>
  </div>
    
  <div class="foreground" slot="foreground">
    {#each sections as section, i}
      <section>
        {#if i === index}
          <div in:fade={{duration: 700 }} out:fade={{duration: 400 }}>
            <h1>{section.title}</h1>
            <p id='left'>{section.content}</p>
            {#if section.imageUrl}
              <img src={section.imageUrl} alt={section.title} class="section-image"/>
            {/if}
          </div>
        {/if}
      </section>
    {/each}
  </div>
</Scroller>

<style>
  section {
    font-family: 'Monaco', monospace;
  }
  .background {
    position: relative;
  }

  .chart-container {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
  }

  .foreground {
    width: 100%;
    margin: 0 auto;
    height: auto;
    position: relative;
    /* background-color: rgb(235, 229, 221); */
  }

  section {
    height: 100vh;
    border: maroon 3px solid;
    text-align: center;
    max-width: 100%; 
    color: black;
    padding: 2em;
    margin: 0 0 0 0;
  }

  #left{
    text-align: center;
    padding-left: 50%;
    margin-top: 20px;
  }
  .section-image{
    max-width: 100%; 
    max-height: 40vh; 
    object-fit: contain;
    padding-left: 50%;
    margin-top: 20px;
  }

  .background {
    position: relative;
  }
  .background > div {
    position: absolute;
    top: 0;
    left: 0;
  }
</style>
