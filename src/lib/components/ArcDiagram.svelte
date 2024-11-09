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
    
    let {
        width = 600,
        height = 600,
        arcHeight = 200,
        nodes,
        links,
    }: { width: number, height: number, nodes: Node[], links: Link[], arcHeight: number } = $props();
    
    let graphContainer: HTMLElement;
    let radiusOffset = $state(10); // Default value
    let svg: d3.Selection<SVGGElement, unknown, null, undefined>; // Store svg reference
    
    let matrix: number[][];
    let x: d3.ScaleBand<string>;
    let y: d3.ScaleBand<string>;
    let color: d3.ScaleSequential<string>;

    // Add these variables for hover state and optimization
    let hoveredSource: number | null = $state(null);
    let hoveredTarget: number | null = $state(null);
    let matrixCells: d3.Selection<any, any, any, any>;
    let horizontalArcs: d3.Selection<any, any, any, any>;
    let verticalArcs: d3.Selection<any, any, any, any>;

    onMount(() => {
        // Arc Diagram with Adjacency Matrix
        const margin = { top: 50, right: 50, bottom: 50, left: 50 };
        const matrixWidth = width - margin.left - margin.right;
        const matrixHeight = height - margin.top - margin.bottom;

        svg = d3.select(graphContainer)
            .append('svg')
            .attr('width', matrixWidth + margin.left + margin.right)
            .attr('height', matrixHeight + margin.top + margin.bottom)
            .append('g')
            .attr('transform', `translate(${margin.left},${margin.top})`);

        // Create adjacency matrix
        matrix = Array(nodes.length).fill(0).map(() => Array(nodes.length).fill(0));
        links.forEach((link: any) => {
            const sourceIndex = nodes.findIndex((n: any) => n.id === link.source);
            const targetIndex = nodes.findIndex((n: any) => n.id === link.target);
            matrix[sourceIndex][targetIndex] = link.value;
            matrix[targetIndex][sourceIndex] = link.value;
        });
        
        x = d3.scaleBand()
            .domain(nodes.map(d => d.id))
            .range([arcHeight, matrixWidth]);

        y = d3.scaleBand()
            .domain(nodes.map(d => d.id))
            .range([arcHeight, matrixHeight]);

        // Create color scale for the matrix
        color = d3.scaleSequential()
            .interpolator(d3.interpolateBlues)
            .domain([0, d3.max(matrix.flat()) || 1]);
        
        // Create matrix cells
        matrixCells = svg.selectAll('.matrix-cell')
            .data(matrix)
            .enter()
            .append('g')
            .selectAll()
            .data((d, i: number) => d.map((value: number, j: number) => ({value, i, j})))
            .enter()
            .append('rect')
            .attr('class', 'matrix-cell')
            .attr('x', (d: {value: number, i: number, j: number}) => x(nodes[d.j].id)!)
            .attr('y', (d: {value: number, i: number, j: number}) => y(nodes[d.i].id)!)
            .attr('width', x.bandwidth())
            .attr('height', y.bandwidth())
            .style('fill', (d: {value: number, i: number, j: number}) => color(d.value))
            .style('stroke', '#fff')
            .on('mouseover', (event: any, d: {value: number, i: number, j: number}) => {
                hoveredSource = d.i;
                hoveredTarget = d.j;
            })
            .on('mouseout', () => {
                hoveredSource = null;
                hoveredTarget = null;
            });

        // Create horizontal arcs
        horizontalArcs = svg.selectAll('.horizontal-arc')
            .data(links)
            .join('path')
            .attr('class', 'arc-path horizontal-arc')
            .attr('d', (link: any) => {
                const sourceIndex = nodes.findIndex((n: any) => n.id === link.source);
                const targetIndex = nodes.findIndex((n: any) => n.id === link.target);
                const x1 = x(nodes[sourceIndex].id)!;
                const x2 = x(nodes[targetIndex].id)!;
                const startX = x1 + x.bandwidth() / 2;
                const endX = x2 + x.bandwidth() / 2;
                const startY = arcHeight;
                const arcRadius = Math.abs(endX - startX) / 2 + radiusOffset;
                return `M ${startX},${startY} A ${arcRadius},${arcRadius} 0 0,1 ${endX},${startY}`;
            })
            .attr('data-source', (d: any) => d.source)
            .attr('data-target', (d: any) => d.target)
            .style('fill', 'none')
            .style('stroke', (d: any) => color(d.value))
            .style('stroke-width', (d: any) => Math.sqrt(d.value))

        // Create vertical arcs
        verticalArcs = svg.selectAll('.vertical-arc')
            .data(links)
            .join('path')
            .attr('class', 'arc-path vertical-arc')
            .attr('d', (link: any) => {
                const sourceIndex = nodes.findIndex((n: any) => n.id === link.source);
                const targetIndex = nodes.findIndex((n: any) => n.id === link.target);
                const y1 = y(nodes[sourceIndex].id)!;
                const y2 = y(nodes[targetIndex].id)!;
                const startYVert = y1 + y.bandwidth() / 2;
                const endYVert = y2 + y.bandwidth() / 2;
                const startXVert = arcHeight;
                const arcRadiusVert = Math.abs(endYVert - startYVert) / 2 + radiusOffset;
                return `M ${startXVert},${endYVert} A ${arcRadiusVert},${arcRadiusVert} 0 0,1 ${startXVert},${startYVert}`;
            })
            .attr('data-source', (d: any) => d.source)
            .attr('data-target', (d: any) => d.target)
            .style('fill', 'none')
            .style('stroke', (d: any) => color(d.value))
            .style('stroke-width', (d: any) => Math.sqrt(d.value))
    });

    // Make the effect react to arc radius offset changes
    $effect(() => {
        console.log('radiusOffset', radiusOffset);
        svg.selectAll('.horizontal-arc')
            .attr('d', (link: any) => {
                const sourceIndex = nodes.findIndex((n: any) => n.id === link.source);
                const targetIndex = nodes.findIndex((n: any) => n.id === link.target);
                const x1 = x(nodes[sourceIndex].id)!;
                const x2 = x(nodes[targetIndex].id)!;
                const startX = x1 + x.bandwidth() / 2;
                const endX = x2 + x.bandwidth() / 2;
                const startY = arcHeight;
                const arcRadius = Math.abs(endX - startX) / 2 + radiusOffset;
                return `M ${startX},${startY} A ${arcRadius},${arcRadius} 0 0,1 ${endX},${startY}`;
            })
        svg.selectAll('.vertical-arc')
            .attr('d', (link: any) => {
                const sourceIndex = nodes.findIndex((n: any) => n.id === link.source);
                const targetIndex = nodes.findIndex((n: any) => n.id === link.target);
                const y1 = y(nodes[sourceIndex].id)!;
                const y2 = y(nodes[targetIndex].id)!;
                const startYVert = y1 + y.bandwidth() / 2;
                const endYVert = y2 + y.bandwidth() / 2;
                const startXVert = arcHeight;
                const arcRadiusVert = Math.abs(endYVert - startYVert) / 2 + radiusOffset;
                return `M ${startXVert},${endYVert} A ${arcRadiusVert},${arcRadiusVert} 0 0,1 ${startXVert},${startYVert}`;
            })
    });
    
    // Optimize the hover effect
    $effect(() => {
        if (!svg || hoveredSource === null || hoveredTarget === null) {
            // Reset all elements to default state
            matrixCells?.style('opacity', 1);
            horizontalArcs?.style('stroke-width', (d: any) => Math.sqrt(d.value));
            verticalArcs?.style('stroke-width', (d: any) => Math.sqrt(d.value));
            return;
        }

        // Create a Set for faster lookups
        const highlightedIndices = new Set([hoveredSource, hoveredTarget]);

        // Update matrix cells
        matrixCells.style('opacity', (d: {i: number, j: number}) => 
            highlightedIndices.has(d.i) || highlightedIndices.has(d.j) ? 1 : 0.1
        );

        // Update arcs
        const updateArcWidth = (link: any) => {
            const sourceIndex = nodes.findIndex(n => n.id === link.source);
            const targetIndex = nodes.findIndex(n => n.id === link.target);
            return (highlightedIndices.has(sourceIndex) || highlightedIndices.has(targetIndex))
                ? Math.sqrt(link.value) * 2
                : Math.sqrt(link.value);
        };

        horizontalArcs.style('stroke-width', updateArcWidth);
        verticalArcs.style('stroke-width', updateArcWidth);
    });
    
</script>

<div class="controls">
    <label>
        Arc Radius Offset:
        <input 
            type="range" 
            bind:value={radiusOffset} 
            min="0" 
            max="50" 
            step="1"
        />
        <span class="value">{radiusOffset}</span>
    </label>
</div>

<div bind:this={graphContainer} class="graph"></div>

<style>
    .controls {
        margin-bottom: 1rem;
        padding: 1rem;
        background: #f5f5f5;
        border-radius: 8px;
    }

    .controls label {
        display: flex;
        align-items: center;
        gap: 1rem;
        font-size: 0.9rem;
    }

    .controls input[type="range"] {
        flex: 1;
    }

    .controls .value {
        min-width: 2rem;
    }

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

    :global(.matrix-cell) {
        transition: opacity 0.2s ease;
    }

    :global(.arc-path) {
        transition: opacity 0.2s ease;
        pointer-events: all;
    }
</style>