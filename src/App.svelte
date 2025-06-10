<script lang="ts">
  import { onMount } from 'svelte';
  import {DataSet, Network} from "vis-network/standalone";
  
  let container;
  let network;

  type Tag = {
    name: string
    description: string
  }

  type Project = {
    name: string,
    description: string,
    tags: string[],
    related: string[]
    image: string,

  }

  const TAG_RIVALS = {name: "Rivals of Aether", description: "Related to the pixel art platform fighter. The game has a great modding scene which I tried to support."}
  const TAG_TOOLS = {name: "Tools I made", description: "Intended to be useful"}

  const projects: Project[] = [
      {
        name: "Rivals Workshop Assistant",
        description: "Probably the the biggest project I've ever made. It plugged into the editor and invisibly ran in the background, and did so many things people get bored before I can list them all. The game's language doesn't allow importing from other files, so it would inject used functions into the code as needed. It would automatically refortmat and export spritesheets every time you save. It allowed writing attack files in roughly half as many lines. Many more things. Nearly no one used it, but a few people did.",
        image: "rivals_workshop_assistant.png",
        tags: [TAG_RIVALS.name, TAG_TOOLS.name],
        related: ["Rivals lib"]
      },

  ]

  const projectData = {
    "projects": {
      "Rivals assistant": {
        "tags": [TAG_RIVALS.name, TAG_TOOLS.name],
        "related": ["Rivals lib"]
      },
      "Rivals lib": {
        "tags": ["Rivals of Aether", "writing"],
      },
      "rockigi": {
        "tags": ["Rivals of Aether", "mods I made"],
        "related": ["b-33-tl"]
      },
      "b-33-tl": {
        "tags": ["Rivals of Aether", "mods I made"],
        "related": []
      },
      "Yetispy": {
        "tags": ["Eternal Card Game", "tools I made"]
      },
      "IsEveryoneGone.com": {
        "tags": ["Dumb lil thing"]
      },
      "ThingFight": {
        "tags": ["Dumb lil thing"]
      },
      "typing.toren.dev": {
        "tags": ["Dumb lil thing", "games I made"]
      },
      "toren.dev": {
        "tags": ["me", "index"]
      },
      "tree.toren.dev": {
        "tags": ["me", "index"]
      },
      "thoughts.toren.dev": {
        "tags": ["me", "blog", "index"],
        "related": ["thoughtbot"]
      },
      "thoughtbot": {
        "tags": ["blog", "tools"],
        "related": []
      },
      "Tabletop teamfight": {
        "tags": ["board games", "games I made"],
        "related": ["tactics.toren.dev"]
      },
      "tactics.toren.dev": {
        "related": []
      },
      "ThirdTime.toren.dev": {
        "tags": ["tools I made"]
      },
      "board game ai blog posts": {
        "tags": ["blog", "board games", "game ai"]
      },
      "lindholm": {
        "tags": ["rip", "overwatch", "tools I made"]
      },
      "llm social deduction game play": {
        "tags": ["experimental", "llms", "game ai"],
      },
      "universal board game ai": {
        "tags": ["experimental", "game ai"],
      }
    }
  };
  
  function convertToVisData(projectData) {
    const nodes = [];
    const edges = [];
    const tagSet = new Set();
    
    const makeId = (text) => text.toLowerCase().replace(/[^a-z0-9]/g, '-').replace(/-+/g, '-').replace(/^-|-$/g, '');
    
    // Collect all unique tags
    Object.values(projectData.projects).forEach(project => {
      if (project.tags) {
        project.tags.forEach(tag => tagSet.add(tag));
      }
    });
    
    // Create project nodes
    Object.keys(projectData.projects).forEach(projectName => {
      const project = projectData.projects[projectName];
      nodes.push({
        id: makeId(projectName),
        label: projectName,
        type: 'project',
        color: { background: '#97c2fc', border: '#2b7ce9' },
        shape: 'box'
      });
    });
    
    // Create tag nodes
    tagSet.forEach(tag => {
      nodes.push({
        id: makeId(tag),
        label: tag,
        type: 'tag',
        color: { background: '#ffc49c', border: '#ff8c42' },
        shape: 'ellipse'
      });
    });
    
    // Create edges
    let edgeId = 0;
    Object.keys(projectData.projects).forEach(projectName => {
      const project = projectData.projects[projectName];
      const projectId = makeId(projectName);
      
      // Project-to-tag edges
      if (project.tags) {
        project.tags.forEach(tag => {
          edges.push({
            id: `edge-${edgeId++}`,
            from: projectId,
            to: makeId(tag),
            color: { color: '#848484' },
            dashes: true
          });
        });
      }
      
      // Project-to-project edges
      if (project.related) {
        project.related.forEach(relatedProject => {
          edges.push({
            id: `edge-${edgeId++}`,
            from: projectId,
            to: makeId(relatedProject),
            color: { color: '#2b7ce9' },
            width: 2
          });
        });
      }
    });
    
    return { nodes, edges };
  }
  
  onMount(async () => {
    const visData = convertToVisData(projectData);
    
    const data = {
      nodes: new DataSet(visData.nodes),
      edges: new DataSet(visData.edges)
    };
    
    const options = {
      nodes: {
        font: { size: 14 },
        margin: 10
      },
      edges: {
        font: { size: 12 },
        smooth: {
          type: 'continuous'
        }
      },
      physics: {
        stabilization: { iterations: 200 },
        barnesHut: {
          gravitationalConstant: -30000,
          centralGravity: 1,
          springLength: 200,
          springConstant: 0.05,
          damping: 0.9
        }
      },
      interaction: {
        hover: true,
        selectConnectedEdges: false
      }
    };
    
    network = new Network(container, data, options);
    
    // Add click handler
    network.on('click', function(params) {
      if (params.nodes.length > 0) {
        const nodeId = params.nodes[0];
        const node = data.nodes.get(nodeId);
        console.log('Clicked node:', node);
      }
    });
  });
</script>

<main>
  <div bind:this={container} class="graph-container"></div>
</main>

<style>
  main {
    padding: 20px;
    font-family: Arial, sans-serif;
  }
  
  h1 {
    color: #333;
    margin-bottom: 20px;
  }
  
  .graph-container {
    width: 100%;
    height: 600px;
    border: 1px solid #ccc;
    border-radius: 8px;
    background: #fafafa;
  }
</style>