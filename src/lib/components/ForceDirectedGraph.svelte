<script lang="ts">
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    type Node = {
        id: string;
        label: string;
        group: string;
    }
    type Link = {
        source: string;
        target: string;
        value: number;
    }
    
    let { width = 600, height = 600, nodes, links }: { width: number, height: number, nodes: Node[], links: Link[] } = $props();
    
    let graphContainer: HTMLElement;

    onMount(() => {
        const svg = d3.select(graphContainer)
            .append('svg')
            .attr('width', width)
            .attr('height', height);

        // Create force simulation
        const forceSimulation = d3.forceSimulation(nodes as d3.SimulationNodeDatum[])
            .force('link', d3.forceLink(links).id((d: any) => d.id))
            .force('charge', d3.forceManyBody().strength(-50))
            .force('center', d3.forceCenter(width / 2, height / 2))
            
        // Add links
        const link = svg.append('g')
            .selectAll('line')
            .data(links)
            .join('line')
            .style('stroke', '#999')
            .style('stroke-opacity', 0.6)
            .style('stroke-width', (d: any) => Math.sqrt(d.value));
        
        // Create color scale for node groups
        const colorScale = d3.scaleOrdinal<string, string>()
            .domain([...new Set(nodes.map(d => d.group))])
            .range(d3.schemeCategory10);

        // Add nodes
        const node = svg.append('g')
            .selectAll('circle')
            .data(nodes)
            .join('circle')
            .attr('r', 5)
            .style('fill', d => colorScale(d.group))
            .call(drag(forceSimulation));

        // Add node labels
        const labels = svg.append('g')
            .selectAll('text')
            .data(nodes)
            .join('text')
            .text((d: any) => d.label)
            .attr('font-size', '8px')
            .attr('dx', 8)
            .attr('dy', 3);
            
        forceSimulation.on('tick', () => {
            link
                .attr('x1', (d: any) => d.source.x)
                .attr('y1', (d: any) => d.source.y)
                .attr('x2', (d: any) => d.target.x)
                .attr('y2', (d: any) => d.target.y);

            node
                .attr('cx', (d: any) => d.x)
                .attr('cy', (d: any) => d.y);

            labels
                .attr('x', (d: any) => d.x)
                .attr('y', (d: any) => d.y);
        });
    });

    // Drag functionality for force-directed graph
    function drag(simulation: d3.Simulation<d3.SimulationNodeDatum, d3.SimulationLinkDatum<d3.SimulationNodeDatum>>) {
        function dragstarted(event: d3.D3DragEvent<any, any, any>) {
            if (!event.active) simulation.alphaTarget(0.3).restart();
            const subject = event.subject as d3.SimulationNodeDatum;
            subject.fx = subject.x;
            subject.fy = subject.y;
        }

        function dragged(event: d3.D3DragEvent<any, any, any>) {
            const subject = event.subject as d3.SimulationNodeDatum;
            subject.fx = event.x;
            subject.fy = event.y;
        }

        function dragended(event: d3.D3DragEvent<any, any, any>) {
            if (!event.active) simulation.alphaTarget(0);
            const subject = event.subject as d3.SimulationNodeDatum;
            subject.fx = null;
            subject.fy = null;
        }

        return d3.drag<any, any>()
            .on('start', dragstarted)
            .on('drag', dragged)
            .on('end', dragended);
    }
</script>

<div bind:this={graphContainer} class="graph"></div>

<style>
    .graph {
        border: 1px solid #ddd;
        border-radius: 8px;
        padding: 1rem;
        background: white;
    }
    
    :global(svg) {
        max-width: 100%;
        height: auto;
    }
</style>