<script lang="ts">
    import { nodes, links } from "$lib/data.json";
    import ForceDirectedGraph from "$lib/components/ForceDirectedGraph.svelte";
	import ArcDiagram from '$lib/components/ArcDiagram.svelte';
</script>

<div class="container">
    <h2>Force-Directed Graph</h2>
    <ForceDirectedGraph
        width={1000}
        height={1000}
        nodes={JSON.parse(JSON.stringify(nodes)).map((node: any) => ({ id: node.id, label: node.name, group: node.grp }))}
        links={JSON.parse(JSON.stringify(links)).map((link: any) => ({ source: link.source, target: link.target, value: link.value }))}
    />
    
    <h2>Adjacency Matrix with Arc Diagram</h2>
    <ArcDiagram
        width={1000}
        height={1000}
        arcHeight={200}
        nodes={JSON.parse(JSON.stringify(nodes)).map((node: any) => ({ id: node.id, label: node.name, group: node.grp }))}
        links={JSON.parse(JSON.stringify(links)).map((link: any) => ({ source: link.source, target: link.target, value: link.value }))}
    />
</div>

<style>
    .container {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 2rem;
        padding: 2rem;
    }

    h2 {
        color: #333;
        text-align: center;
    }
</style>