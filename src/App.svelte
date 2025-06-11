<script lang="ts">
    import {onMount} from 'svelte';
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
        tags: Tag[],
        related?: string[]
        image: string,
        link?: string,
    }

    const TAG_RIVALS = {
        name: "Rivals of Aether",
        description: "Related to the pixel art platform fighter. The game has a great modding scene which I tried to support."
    }
    const TAG_TOOLS = {name: "Tools I made", description: "Intended to be useful."}
    const TAG_WRITING = {
        name: "Stuff I wrote",
        description: "I'm always surprised when I'm able to communicate something clearly."
    }
    const TAG_MODS = {
        name: "Game mods I made",
        description: "Making content for a game. This is fun and you should try it some time. Easy for board games."
    }
    const TAG_ETERNAL = {
        name: "Eternal Card Game",
        description: "An online card game I used to play. A whole lot less of a grindy pay to win than most."
    }
    const TAG_DUMB = {
        name: "Dumb Lil Thing",
        description: "Lower your expectations"

    }
    const TAG_RIP = {
        name: "RIP",
        description: "Used to work but time killed it. Might be fixable if important enough."
    }
    const TAG_GAME = {
        name: "Games I Made",
        description: "Intended to be fun."
    }
    const TAG_ME = {
        name: "Me",
        description: "Work where I am the subject matter."
    }
    const TAG_INDEX = {
        name: "Index",
        description: "For finding other things, like this page you're on now."
    }
    const TAG_BOARD_GAME = {
        name: "Board Gamey",
        description: "Physical games are far faster to prototype and mess around with."
    }
    const TAG_GAME_AI = {
        name: "Game AI",
        description: "Games were made to be lost at against unbeatable ai."
    }
    const TAG_OVERWATCH = {
        name: "Overwatch",
        description: "Once upon a time, was a decent game."
    }
    const TAG_EARLY_WIP = {
        name: "Early WIP",
        description: "No finished products here. May or may not be completed later."
    }
    const TAG_LLM = {
        name: "LLM",
        description: "Stuff centered around using llms. Usually requires a lot of wrapping peripherals."
    }


    const projects: Project[] = [
        {
            name: "Rivals Workshop Assistant",
            description: "Probably the the biggest project I've ever made. It plugged into the editor and invisibly ran in the background, and did so many things people get bored before I can list them all. The game's language doesn't allow importing from other files, so it would inject used functions into the code as needed. It would automatically refortmat and export spritesheets every time you save. It allowed writing attack files in roughly half as many lines. Many more things. Nearly no one used it, but a few people did.",
            image: "rivals_workshop_assistant.png",
            tags: [TAG_RIVALS, TAG_TOOLS],
            related: ["Rivals lib"],
            link:"https://www.rivalslib.com/assistant/"
        },
        {
            name: "Rivals Workshop Community Library",
            description: "There were a lot of new modders for Rivals of Aether, and they all had to learn programming, pixel art and animation, and character design from scratch. I wrote up a guided tutorial on all those things, with information from expert interviews in places. Sadly I did a stupid and used some live image links, so there are now some broken images in the art section.",
            image: "rivals_lib.png",
            tags: [TAG_RIVALS, TAG_WRITING],
            link: "https://www.rivalslib.com"
        },
        {
            name: "Rockigi",
            description: "The only full character I made for Rivals that got past a silly concept phase. It's 0% original, being based off of TDude's Rocky, and melee Luigi. I'd say it's a lot of fun, and uses some neat mobility tricks and trap options to carry the opponent around.",
            image: "rockigi.jfif",
            tags: [TAG_RIVALS, TAG_MODS],
            related: ["b-33-tl"],
            link: "https://steamcommunity.com/sharedfiles/filedetails/?id=2820351983",
        },
        {
            name: "B-33-TL",
            description: "A simple little character made with talented friend Frtoud and beautiful wife Mabel for a competition. Bizarrely we won. The character is based around magnetically manipulating a floating ball (robot dung?) and there's all sorts of clever things you can do with it.",
            image: "b33tl.jfif",
            tags: [TAG_RIVALS, TAG_MODS],
            link: "https://steamcommunity.com/sharedfiles/filedetails/?id=3290301236",
        },
        {
            name: "Yetispy",
            description: "Seems to be dead and idk if I want to fix the config given no one is using it now. I used to play a digital card game, and sometimes you buy cards with your fake in game money. What set to buy? Past me would be damned if he purchased suboptimally, so he made this very complicated tool that takes into account card popularity trends, rarities and resell prices, your current collection, etc, to calculate the optimal collection-improvement per gold. It keeps itself updated as the game gets new content",
            image: "yetispy.png",
            tags: [TAG_TOOLS, TAG_RIP],
            link: "https://yetispy.com",
        },
        {
            name: "Is Everyone Gone Dot Com",
            description: "For when you wake up and no one's around and its quiet outside.",
            image: "iseveryonegonedotcom.png",
            tags: [TAG_DUMB],
            link: "https://iseveryonegone.com"
        },
        {
            name: "ThingFight",
            description: "Real time ELO voting game. Children Vs. Half Life 2? Petting a cat Vs. Christopher Lloyd?",
            image: "thingfight.png",
            tags: [TAG_DUMB, TAG_GAME],
            link: "https://thingfight.toren.dev",
        },
        {
            name: "Typing.toren.dev",
            description: "Warioware typing game. Has a lot of potential to be prettier and more interesting. Originally I wanted questions with llm answer judging but that requires a server and leaks money. Could have users give an api key, but no one is going to do that.",
            image: "typing.png",
            tags: [TAG_DUMB, TAG_GAME],
            link: "https://typing.toren.dev"
        },
        {
            name: "Toren.dev",
            description: "My portfolio site. Not sure anyone's seen it, but you have to buy yourname.dev and put it at the top of your resume. A bit motion sickness inducing but I like it.",
            image: "toren.jpg",
            tags: [TAG_ME, TAG_INDEX],
            link: "https://toren.dev"
        },
        {
            name: "This Page",
            description: "What you're looking at. It's a graph of everything I've done so everything's indexed in one place.",
            image: "thingsimade.png",
            tags: [TAG_ME, TAG_INDEX],
            link: "https://thingsimade.toren.dev"
        },
        {
            name: "Thoughts.toren.dev",
            description: "Microblog. I spam my friend pretty severely and end up trying to use discord search to go the idea heap sometimes. Now I have a bot repost my spam to a blog.",
            image: "thoughts.png",
            tags: [TAG_ME, TAG_INDEX, TAG_WRITING],
            link: "https://thoughts.toren.dev"
        },
        {
            name: "ThoughtBot",
            description: "A bot for reposting your discord posts onto a blog. Mostly designed for personal use but go ahead.",
            image: "thoughtbot.png",
            tags: [TAG_TOOLS],
            link: "https://github.com/Qazzquimby/thought-blog/tree/master/thought-bot"
        },
        {
            name: "Tabletop Teamfight",
            description: "I find wargames too slow and I really like figuring out synergies and counters, so I made a streamlined crossover tactics boardgame where overwatch bastion can ride on lancer lancaster and get spammed out by tf2 demoman and dota earthshaker. Stalled due to tabletop simulator technical difficulties, but pretty okay game.",
            image: "tabletopteamfight.webp",
            tags: [TAG_GAME, TAG_BOARD_GAME],
            link: "https://steamcommunity.com/sharedfiles/filedetails/?id=3096969105&searchtext=teamfight",
            related: ["Tactics.toren.dev"]
        },
        {
            name: "Tactics.toren.dev",
            description: "Where I host the rules and assets for Tabletop Teamfight",
            image: "tactics.png",
            tags: [],
            link: "https://tactics.toren.dev/rules"
        },
        {
            name: "ThirdTime",
            description: "A pomodoro variant that doesn't tell you when to take your breaks. It tracks how long you've worked and allocates a ratio of that as rest time.",
            image: "thirdtime.png",
            tags: [TAG_TOOLS],
            link: "https://thirdtime.toren.dev"
        },
        {
            name: "Board Game AI Explainer",
            description: "I think board game AI is really interesting and wanted to see if I could make it understandable for people with no relevant background. Will hopefully get more posts and also document my work on universal board game ai.",
            image: "boardgameaiexplainer.png",
            tags: [TAG_WRITING, TAG_BOARD_GAME, TAG_GAME_AI],
            link: "https://thoughts.toren.dev/blog/universal-board-game-ai-1/",
            related: ["Universal Board Game AI"]
        },
        {
            name: "Lindholm",
            description: "When overwatch wasn't garbage I ran the world's only 24/7 fully automated server, thanks to this bot that looked at the screen and clicked the buttons needed to balance teams, add bots when the player count was low, skip post game, etc.",
            image: "lindholm.png",
            tags: [TAG_RIP, TAG_OVERWATCH],
            related: ["Unreal300 Server"],
            link: "https://github.com/Qazzquimby/Lindholm"
        },
        {
            name: "Unreal300 Server",
            description: "Fast and lethal and very finely tuned to reward skill and cool plays. For a while it had a community of around 100 members. I have clips somewhere, but there's nothing live to link to anymore.",
            image: "unreal300.png",
            tags: [TAG_RIP, TAG_OVERWATCH, TAG_MODS]
        },
        {
            name: "LLM Social Deduction Game Play",
            description: "Setting things up so llms can play werewolf and blood on the clocktower. Decent at werewolf actually. Obviously managing a long and highly complex gamestate without hallucinating or falling for overconfidence is a problem, but it turns out just handling a many user long conversation is a serious roadblock.",
            image: "llmsocial.png",
            link: "https://github.com/Qazzquimby/LlmSocialDeduction",
            tags: [TAG_EARLY_WIP, TAG_LLM, TAG_GAME_AI]
        },
        {
            name: "LLM Long Term Memory & Text Adventure Play",
            description: "Playing pokemon tests pre-existing game knowledge and vision capabilities too much. I set up a harness for llms to play text adventures and used it to test a nuanced long term memory system. Blocker for improvement is how much it costs, since it needs to be a good llm handling very large scale tasks for memory to become a factor.",
            image: "llmmemory.png",
            link: "https://github.com/Qazzquimby/llm-long-term-memory",
            tags: [TAG_LLM, TAG_GAME_AI]
        },
        {
            name: "Universal Board Game AI",
            description: "Any game state can be stored as a homogeneous graph. Zero-MCTS based algorithms can learn any game given self play. A custom game engine can generate the models and everything they need from the rules definition.",
            image: "universalai.png",
            link: "https://github.com/Qazzquimby/boardgame-ai-engine",
            tags: [TAG_EARLY_WIP, TAG_GAME_AI]
        }





    ]


    const _projectData = {
        "projects": {
            "universal board game ai": {
                "tags": ["experimental", "game ai"],
            }
        }
    };

    function wrapTitle(text: string) {
        const container = document.createElement("span");
        container.innerText = text;
        container.style = "white-space: normal; background: white; max-width: 2rem;"
        return container;
    }

    function convertToVisData(projects: Project[]) {
        const nodes = [];
        const edges = [];
        const tagSet: Set<Tag> = new Set();

        const makeId = (text: string) => text.toLowerCase().replace(/[^a-z0-9]/g, '-').replace(/-+/g, '-').replace(/^-|-$/g, '');

        // Collect all unique tags
        projects.forEach(project => {
            if (project.tags) {
                project.tags.forEach(tag => tagSet.add(tag));
            }
        });

        // Create project nodes
        projects.forEach(project => {
            nodes.push({
                id: makeId(project.name),
                label: project.name,
                title: wrapTitle(project.description),
                type: 'project',
                color: {background: '#97c2fc', border: '#2b7ce9'},
                shape: 'image',
                image: `public/project_images/${project.image}`,
                size: 50,
                mass: 2.5,
                link: project.link

            });
        });

        // Create tag nodes
        tagSet.forEach(tag => {
            nodes.push({
                id: makeId(tag.name),
                label: tag.name,
                title: tag.description,
                type: 'tag',
                color: {background: '#59cbf4', border: '#149ae8'},
                font: {
                    color: "#000",
                },
                shape: 'box'
            });
        });

        // Create edges
        let edgeId = 0;
        projects.forEach(project => {
            const projectId = makeId(project.name);

            // Project-to-tag edges
            if (project.tags) {
                project.tags.forEach(tag => {
                    edges.push({
                        id: `edge-${edgeId++}`,
                        from: projectId,
                        to: makeId(tag.name),
                        color: {color: '#f3ecec'},
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
                        color: {color: '#2b7ce9'},
                        width: 2
                    });
                });
            }
        });

        return {nodes, edges};
    }

    onMount(async () => {
        const visData = convertToVisData(projects);

        const data = {
            nodes: new DataSet(visData.nodes),
            edges: new DataSet(visData.edges)
        };

        const options = {
            autoResize: true,
            nodes: {
                font: {
                    size: 32,
                    color: "#FFF"
                },
                size: 30,
            },
            edges: {
                font: {size: 12},
            },
            physics: {
                stabilization: true,
                barnesHut: {
                    gravitationalConstant: -30000,
                    centralGravity: 3.0,
                    springLength: 150,
                    springConstant: 0.05,
                    damping: 0.1
                }
            },
            interaction: {
                hover: true,
                selectConnectedEdges: true
            }
        };

        network = new Network(container, data, options);

        // Add click handler
        network.on('click', function (params) {
            if (params.nodes.length > 0) {
                const nodeId = params.nodes[0];
                const node = data.nodes.get(nodeId);
                if(node.link){
                    window.open(node.link, '_blank').focus();
                }

            }
        });
    });
</script>

<main>
    <div bind:this={container} class="graph-container"></div>
</main>

<style>
    main {
        height: 100%
    }

    .graph-container {
        width: 100%;
        height: 100%;
        background-color: #222222;
    }


</style>