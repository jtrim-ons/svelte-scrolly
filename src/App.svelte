<script>
	import { setContext, onMount } from "svelte";
	import { runAction, getData, setColors } from "./utils.js";
	import { themes, regions, colors, datakeys } from './config.js';
	import { ScatterChart } from "@onsvisual/svelte-charts";
	import ONSHeader from "./ONSHeader.svelte";
	import ONSFooter from "./ONSFooter.svelte";
	import Header from "./Header.svelte";
	import Section from "./Section.svelte";
	import Media from "./Media.svelte";
	import Scroller from "./Scroller.svelte";
	import Filler from "./Filler.svelte";
	import Divider from "./Divider.svelte";
	import Map from "./Map.svelte";

	// STYLE CONFIG
	// Set theme globally (options are 'light' or 'dark')
	let theme = "light";
	setContext("theme", theme);
	setColors(themes, theme);

	// SCROLLYTELLING CONFIG
	// Config
	const threshold = 0.9;
	// State
	let chartSectionIndex = 0;
	let mapSectionIndex = 0;

	// CHART CODE
	// Config
	const categories = regions.map((d) => d.code);
	const url = "https://raw.githubusercontent.com/bothness/geo-data/main/csv/imd-values-2019.csv";
	const catKey = "region_code";
	// State
	let data;
	let places;
	let selected;
	let xKey = "income";
	let yKey = "employment";

	getData(url)
		.then((result) => (data = result))
		.then(() => {
			let codes = [];
			let array = [];
			data.forEach((d) => {
				if (!codes.includes(d.lad_code)) {
					codes.push(d.lad_code);
					array.push({
						code: d.lad_code,
						name: d.lad_name,
					});
				}
		});
		array.sort((a, b) => a.name.localeCompare(b.name));
		places = array;
	});
	
	// Actions for CHART scroller
	const chartActions = [
		() => {
			xKey = "income";
			yKey = "employment";
		},
		() => {
			xKey = "income";
			yKey = "health";
		},
		() => {
			xKey = "housing";
			yKey = "health";
		}
	];
	
	// Reactive code to trigger CHART actions
	$: {runAction(chartActions[+chartSectionIndex]);}
	
	// MAP CODE
	// Config
	const mapstyle = 'https://bothness.github.io/ons-basemaps/data/style-omt.json';
	const mapbounds = {
		ew: [[-5.816, 49.864], [1.863, 55.872]],
		fareham: [[-1.2280, 50.8368], [-1.1650, 50.8699]],
		newport: [[-3.0682, 51.5448], [-2.9170, 51.6258]]
	};
	// State
	let map = null;

	// Actions for MAP scroller
	const mapActions = [
		() => { map.fitBounds(mapbounds.ew) },
		() => { map.fitBounds(mapbounds.fareham) },
		() => { map.fitBounds(mapbounds.newport) }
	];
	
	// Reactive code to trigger MAP actions
	$: if (map) {
		runAction(mapActions[+mapSectionIndex]);
	}

</script>

<ONSHeader filled={true} />

<Header bgimage="./img/bg-dark.jpg" bgfixed={true} theme="dark">
	<h1 class="text-shadow">This is the title of the article</h1>
	<p class="inset-medium text-big text-shadow">
		This is a short text description of the article that might take up a couple of lines
	</p>
	<div class="text-shadow" style="margin-top: 48px;">
		Scroll to begin<br />
		<img src="./img/scroll-down-white.svg" class="svg-icon bounce" alt="down arrow"/>
	</div>
</Header>

<Filler theme="dark">
	<p class="text-big">
		This is a large text caption centred on a full-screen page
	</p>
</Filler>

<Section>
	<h2>This is a section title</h2>
	<p>
		This is a short paragraph of text to demonstrate the standard column width 
		font size and line spacing of the template.
	</p>
	<p>
		This is a second short paragraph of text to demonstrate the size of the
		paragraph spacing in the template.
	</p>
	<blockquote class="text-indent">
		"This is an example of a large embedded quotation."&mdash;Person
	</blockquote>
	<p>
		Below is a grid that could contain charts or any other kind of visual media.
		The grid can fit in a standard, wide or full-width column, and the media
		width itself can be narrow (min 200px), medium (min 300px), wide (min 500px)
		or full-width. The grid is responsive, and will re-flow on smaller screens. 
	</p>
</Section>

<Media
	col="medium"
	grid="medium"
	caption='This is an optional caption for the above media. It can contain HTML code and <a href="#">hyperlinks</a>, and wrap onto multiple lines.'>
	<div class="media">media</div>
	<div class="media">media</div>
	<div class="media">media</div>
	<div class="media">media</div>
</Media>

<Divider />

<Section>
	<h2>This is a dynamic chart section</h2>
	<p>
		The chart below will respond to the captions as you scroll down. 
		The mode is set to splitscreen, with captions on the left on larger screens.
	</p>
</Section>

<Scroller {threshold} bind:index={chartSectionIndex} splitscreen={true}>
	<div slot="background">
		<figure>
			<div class="col-wide height-full middle">
				{#if data && xKey && yKey}
				<div class="chart">
					<ScatterChart diameter={3} {data} {xKey} {yKey} {catKey} {colors} {categories} {selected} />
				</div>
				{/if}
			</div>
		</figure>
	</div>

	<div slot="foreground">
		<section>
			<div class="col-medium">
				<p>There is a strong correlation between the IMD mesasures for <span class="em em-muted">income</span> 
					and <span class="em em-muted">employment</span>.</p>
			</div>
		</section>
		<section>
			<div class="col-medium">
				<p>There is a strong correlation between the IMD mesasures for <span class="em em-muted">income</span> and 
					<span class="em em-muted">health</span>.</p>
			</div>
		</section>
		<section>
			<div class="col-medium">
				<p>There is a weak correlation between the IMD mesasures for <span class="em em-muted">housing</span> and 
					<span class="em em-muted">health</span>.</p>
			</div>
		</section>
		<section>
			<div class="col-medium">
				<h3>Explore the data</h3>
				{#if data}
				<nobr>
					<span class="label-block">X Axis</span>
					<select bind:value={xKey}>
						{#each Object.keys(datakeys) as key}
							<option value={key}>{datakeys[key]}</option>
						{/each}
					</select>
				</nobr>
				<nobr>
					<span class="label-block">Y Axis</span>
					<select bind:value={yKey}>
						{#each Object.keys(datakeys) as key}
							<option value={key}>{datakeys[key]}</option>
						{/each}
					</select>
				</nobr>
				{#if places}
				<nobr>
						<span class="label-block">District</span>
						<!-- svelte-ignore a11y-no-onchange -->
						<select bind:value={selected}>
							<option value={null}>All</option>
							{#each places as place}
								<option value={{ value: place.code, col: 'lad_code' }}>
									{place.name}
								</option>
							{/each}
						</select>
					</nobr>
				{/if}
				{/if}
			</div>
		</section>
	</div>
</Scroller>

<Divider />

<Section>
	<h2>This is a full-bleed media demo</h2>
	<p>
		Below is an example of a media grid where the column with is set to "full"
		and the grid width is set to default (100%).
	</p>
</Section>

<Media col="full" height={600} caption="This is an optional caption for the above media.">
	<div class="media">full-bleed media</div>
</Media>

<Divider />

<Section>
	<h2>This is a dynamic map section</h2>
	<p class="mb">
		The map below will respond to the captions as you scroll down. 
		The mode is default, with captions over the map on any screen size.
	</p>
</Section>

<Scroller {threshold} bind:index={mapSectionIndex}>
	<div slot="background">
		<figure>
			<div class="col-full height-full">
				<Map style={mapstyle} bind:map={map} />
			</div>
		</figure>
	</div>

	<div slot="foreground">
		<section>
			<div class="col-medium">
				<p>This is a map zoomed to the extents of <span class="em em-muted">England and Wales</span>.</p>
			</div>
		</section>
		<section>
			<div class="col-medium">
				<p>This is where <span class="em em-muted">Fareham, Hampshire</span> is on the map.</p>
			</div>
		</section>
		<section>
			<div class="col-medium">
				<p>This is where <span class="em em-muted">Newport, Gwent</span> is on the map.</p>
			</div>
		</section>
	</div>
</Scroller>

<Divider />

<Section>
	<h2>This is a wide media grid demo</h2>
	<p>
		Below is an example of a media grid where the column width is set to "wide" 
		and the grid width is set to "narrow".
	</p>
</Section>

<Media
	col="wide"
	grid="narrow"
	caption="This is an optional caption for the above media.">
	<div class="media">media</div>
	<div class="media">media</div>
	<div class="media">media</div>
	<div class="media">media</div>
</Media>

<Section>
	<p>
		This is a final paragraph of text to end the demo article layout.
	</p>
</Section>

<ONSFooter />

<style>
	/* Styles specific to elements within the demo */
	.svg-icon {
		width: 48px;
		height: 48px;
	}
	.bounce {
		animation-duration: 2s;
		animation-iteration-count: infinite;
		animation-name: bounce;
		animation-timing-function: ease;
	}
	@keyframes bounce {
		0%   { transform: translateY(10px); }
		30%  { transform: translateY(-10px); }
		50%  { transform: translateY(10px); }
		100% { transform: translateY(10px); }
	}
	.label-block {
		display: inline-block;
		text-align: right;
		width: 80px;
	}
	select {
		width: 210px;
	}
	.chart {
		margin-top: 45px;
		height: 100vh;
		width: calc(100% - 5px);
	}
	/* The properties below make the media DIVs grey, for visual purposes in demo */
	.media {
	  background-color: #f0f0f0;
	  display: -webkit-box;
	  display: -ms-flexbox;
	  display: flex;
	  -webkit-box-orient: vertical;
	  	-webkit-box-direction: normal;
	      -ms-flex-flow: column;
	          flex-flow: column;
	  -webkit-box-pack: center;
	      -ms-flex-pack: center;
	          justify-content: center;
	  text-align: center;
	  color: #aaa;
  }
</style>
