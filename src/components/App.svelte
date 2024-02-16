<script>
    import * as d3 from 'd3';

    import bar_nodes from './f_bars.json'
    import bar_edges from './f_bar_links.json'
    import { select } from 'd3-selection';

    import {onMount} from 'svelte';

    let svg;

    // onMount just does all this stuff as soon as the webpage is loaded. A lot of things seemed to work better when I put
    // them in here but I also don't really know what I'm doing
    onMount(() => {
        const width = 1200;
        const height = 1000;

        const links = bar_edges.map(d => ({...d})); // I don't actually know what this does but it does seem essential
        const nodes = bar_nodes.map(d => ({...d}));

        // Makes the graph look pretty and move around nicely
        const simulation = d3.forceSimulation(nodes)
            .force("link", d3.forceLink(links).id(d => d.id))
            .force("charge", d3.forceManyBody())
            .force("center", d3.forceCenter(width / 3, height / 3))
            .on("tick", ticked);

        // Creates the graphics window inside the div
        svg = d3.select('svg')
            .attr("width", width)
            .attr("height", height)
            .attr("style", "max-width: 100%; height: auto;");

        // Makes all the edges
        const link = svg.append("g")
            .attr("stroke", "#999")
            .attr("stroke-opacity", 0.6)
            .selectAll()
            .data(links) // Contain the extra data from the f_bar_links.json file
            .join("line")
            .attr("stroke-width", d => Math.sqrt(d.value)); // Width of line proportional to number of mutual connections

        // Makes all the nodes
        const node = svg.append("g")
            .attr("stroke", "#000") // Outline
            .attr("fill", '#fff') // Fill Color
            .attr("stroke-width", 1.5)
            .selectAll()
            .data(nodes) // Contain the extra data from the f_bars.json file (very helpful because I can just add more stuff to the dataframe and make a new f_bars.json file)
            .join("circle")
            .attr("r", 7)
            .on("mouseenter", tooltip_in) // When we hover, call the tooltip_in function
            .on("mouseout", tooltip_out) // When we leave, call the tooltip_out function

        // Overarching function for dragging nodes
        node.call(d3.drag()
                .on("start", dragstarted)
                .on("drag", dragged)
                .on("end", dragended));

        // Makes movement "continuous" for all nodes (I think?)
        function ticked() {
            link
                .attr("x1", d => d.source.x)
                .attr("y1", d => d.source.y)
                .attr("x2", d => d.target.x)
                .attr("y2", d => d.target.y);

            node
                .attr("cx", d => d.x)
                .attr("cy", d => d.y);
        }

        // Start drag
        function dragstarted(event) {
            if (!event.active) simulation.alphaTarget(0.3).restart();
            event.subject.fx = event.subject.x;
            event.subject.fy = event.subject.y;
            event.sourceEvent.srcElement.style.fill = 'red';
        }

        // Actually Moving Nodes
        function dragged(event) {
            event.subject.fx = event.x;
            event.subject.fy = event.y;
        }

        // End Drag. Pretty glitchy with the colors. If you let go of a node over the tooltip or out of the svg bounds,
        // it'll just stay red.
        function dragended(event) {
            event.sourceEvent.srcElement.style.fill = 'white';
            if (!event.active) simulation.alphaTarget(0);
            event.subject.fx = null;
            event.subject.fy = null;
        }

        // The tooltip is actually just one div tag that changes position, hides itself, and changes text based on the user's
        // behavior
        let tooltip = d3.select("body")
            .append("div") 
            .style("position", "absolute")
            .style("visibility", "hidden")
            .style("background-color", "white")
            .attr("class", "tooltip")

        // This is called when you hover over a node. It changes the text to match that node and appears wherever you're hovering.
        function tooltip_in(event, d) { 
                console.log('Entered Node')
                return tooltip
                    .html("<h4>" + d.name + "</h4>")
                    .style("visibility", "visible")
                    .style("top", (event.pageY + 10) + "px")
                    .style("left", (event.pageX + 10) + "px");
            }

        // This is meant to hide the tooltip when you leave a node. It's been buggy, but it's a little buggy even in the
        // examples I was looking at online so it might just be like that 
        function tooltip_out() {
                console.log('Left node')
                return tooltip
                .transition()
                .delay(2000)
                .style("visibility", "hidden");
        }
    });
</script>

<div>
    Network Visualization of Jazz Clubs in Downtown New Orleans by Mutual Yelp Reviews
</div>
<div class='centered'>
    <svg class='network' viewBox="190 150 390 350">
    </svg>
</div>

<style>
    @import url("https://fonts.googleapis.com/css2?family=Nunito:wght@300;400;700&display=swap");

    :root {
        --color-bg: #ffffff;
        --color-outline: #c2c2c2;
        --color-shadow: hsl(0, 0%, 0%, 0.1);
        --color-text: #3f4252;
        --color-bg-1: hsla(0, 0%, 0%, 0.2);
        --color-shadow-1: hsl(0, 0%, 96%);
    }

    *,
    *::before,
    *::after {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
    }

    div {
        font-family: sans-serif
    }

    svg {
        max-width: 100%;
        max-height: 100%;
        border: 1px solid black;
        background-color: '#B2AC88';
    }

    :global(.tooltip) {
        fill: white;
        font-family: sans-serif;
        padding-left: 5px;
        padding-right: 5px;
        border: 1px solid grey;
        border-radius: 10px;
    }

    .centered {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        background-color: gray;
        padding: 10px;
        border: 1px solid black;
    }
</style>
