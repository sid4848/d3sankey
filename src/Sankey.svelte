<!--
	@component
	Generates an SVG Sankey chart using [d3-sankey](https://github.com/d3/d3-sankey).
 -->
 <script>
	import { getContext } from 'svelte';
	import * as Sankey from 'd3-sankey';

	const { data, width, height } = getContext('LayerCake');

	/** @type {Function} [colorLinks=d => 'rgba(0, 0, 0, .2)'] – A function to return a color for the links. */
	export let colorLinks = d => 'rgba(0, 0, 0, 0.2)';

	/** @type {Function} [colorNodes=d => '#333'] – A function to return a color for each node. */
	export let colorNodes = d => '#333';

	/** @type {Function} [colorText=d => '#263238'] – A function to return a color for each text label. */
	export let colorText = d => 'white';

	/** @type {Number} [nodeWidth=5] – The width of each node, in pixels, passed to [`sankey.nodeWidth`](https://github.com/d3/d3-sankey#sankey_nodeWidth). */
	export let nodeWidth = 5;

	/** @type {Number} [nodePadding=10] – The padding between nodes, passed to [`sankey.nodePadding`](https://github.com/d3/d3-sankey#sankey_nodePadding). */
	export let nodePadding = 10;

	/** @type {Function} [linkSort=null] – How to sort the links, passed to [`sankey.linkSort`](https://github.com/d3/d3-sankey#sankey_linkSort). */
	export let linkSort = null;

	/** @type {Function} [nodeId=d => d.id] – The ID field accessor, passed to [`sankey.nodeId`](https://github.com/d3/d3-sankey#sankey_nodeId). */
	export let nodeId = d => d.id;

	/** @type {Function} [nodeAlign=d3.sankeyLeft] – An alignment function to position the Sankey blocks. See the [d3-sankey documentation](https://github.com/d3/d3-sankey#alignments) for more. */
	export let nodeAlign = Sankey.sankeyLeft;

	$: sankey = Sankey.sankey()
		.nodeAlign(nodeAlign)
		.nodeWidth(nodeWidth)
		.nodePadding(nodePadding)
		.nodeId(nodeId)
		.size([$width, $height])
		.linkSort(linkSort);

	$: sankeyData = sankey($data);

	$: link = Sankey.sankeyLinkHorizontal();

	$: fontSize = $width <= 320 ? 8 : 12;

    let highlightLinkIndexes = [];
</script>

<style>
	text {
		pointer-events: none;
	}
</style>

<g class="sankey-layer">
	<g class='link-group'>
		{#each sankeyData.links as l, d (`link-${d}`)}
			<!-- svelte-ignore a11y-mouse-events-have-key-events -->
			<path
                key={`link-${d}`}
				d={link(l)}
				fill='none'
				stroke={colorLinks(l)}
				opacity={highlightLinkIndexes.includes(d) ? 1 : 0.25}
				stroke-width={l.width}
                on:mouseover={(e) => {
                    highlightLinkIndexes = [d];
                    // const label = document.getElementById(`label-${d}`);
                    // if (label) {
                    //     label.style.opacity = 1;
                    // }
                  }}
                  on:mouseout={(e) => {
                    highlightLinkIndexes = [];
                    // const label = document.getElementById(`label-${d}`);
                    // if (label) {
                    //     label.style.opacity = 1;
                    // }
                  }}
                />
                <g>
                    <!-- Add label for the link value (date and time) -->
                    {#if highlightLinkIndexes.includes(d)}
                    <text
                        id={`label-${d}`}
                        x={(l.source.x1 + l.target.x0) / 2 + 70}
                        y={(l.y0 + l.y1) / 2 - 15 }
                        dy="0.35em"
                        style="font-size: 10px; fill: white; text-anchor: middle; opacity: 1; font-weight:600">
                        {l.file_name} <!-- Format date and time -->
                    </text>
                    <text
                        id={`label-${d}`}
                        x={(l.source.x1 + l.target.x0) / 2 + 70}
                        y={(l.y0 + l.y1) / 2 }
                        dy="0.35em"
                        style="font-size: 10px; fill: white; text-anchor: middle; opacity: 1; font-weight:600">
                        {l.file_hash} <!-- Format date and time -->
                    </text>
                    <text
                        id={`label-${d}`}
                        x={(l.source.x1 + l.target.x0) / 2 + 70}
                        y={(l.y0 + l.y1) / 2 + 15}
                        dy="0.35em"
                        style="font-size: 10px; fill: white; text-anchor: middle; opacity: 1; font-weight:600">
                        {new Date(l.timestamp).toLocaleString()} <!-- Format date and time -->
                    </text>
                    {/if}
                </g>
		{/each}
	</g>
	<g class='rect-group'>
		{#each sankeyData.nodes as n, i (`react-${i}`)}
		{console.log(n)}
			<!-- svelte-ignore a11y-mouse-events-have-key-events -->
			<rect
                id={`rect-${i}`}
				x={n.x0}
				y={n.y0}
				height={n.y1 - n.y0}
				width={n.x1 - n.x0 + 3}
				fill={colorNodes(n)} 
                on:mouseover={(e) => {
                    highlightLinkIndexes = [...n.sourceLinks.map((l) => l.index), ...n.targetLinks.map((l) => l.index)];
                    
                  }}
                  on:mouseout={(e) => {
                    highlightLinkIndexes = [];
                  }} 
                />
			<text
				x={n.x0 < $width / 4 ? n.x1 + 6 : n.x0 - 6}
				y={(n.y1 + n.y0) / 2}
				dy="{(fontSize / 2) - 2}"
				style="fill: {colorText(n)};
							font-size: {fontSize}px;
                            font-weight: 500;
							text-anchor: {n.x0 < $width / 4 ? 'start' : 'end'};">
				{n.id}
			</text>
            <!-- <text
				x={n.x0 < $width / 4 ? n.x1 + 6 : n.x0 - 6}
				y={(n.y1 + n.y0) / 2 + 20}
				dy="{(fontSize / 2) - 2}"
				style="fill: {colorText(n)};
							font-size: {fontSize}px;
                            font-weight: 500;
							text-anchor: {n.x0 < $width / 4 ? 'start' : 'end'};">
				{n.id}, bhiku
			</text> -->
		{/each}
	</g>
</g>
