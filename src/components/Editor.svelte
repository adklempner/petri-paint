<script>
    let container, dragItem;
    let currentX, currentY, initialX, initialY;
    let xOffset = 0, yOffset = 0;
    let mouseX = 0, mouseY = 0;

    let toolBox = {x: 500, y: -200}
    let tools = [
        {type: "place", label: "O", id: 1},
        {type: "transition", label: "|", id: 2},
        {type: "arc", label: "/", id: 3},
        {type: "move", label: "X", id: 4},
        {type: "add", label: "+", id: 5},
        {type: "sub", label: "-", id: 6},
        {type: "delete", label: "\u{1F5D1}", id: 6},
    ]

    let selectedTool = "move";


    function selectTool(e) {
        selectedTool = e.target.dataset.type;
    }


    let items = [
        { type: "place", id: 1, value: 1, name: "Init", x: -200, y: 0 },
        { type: "place", id: 2, value: 0, name: "Balances Loaded", x: 200, y: 0 },
        { type: "transition", id: 3, value: 0, name: "Load Balances", x: 0, y: -50 }
    ];
    
    let nextId = 3;

    let activeItems = [];
    let active = false, activeItem = null;

    let arcFrom = null, arcTo = null;
    let arcs = [
        { type: "arc", id: 1, from: 0, to: 2, value: 1}
    ]
    let nextArcId = 1;

    let sizeConfig = {
        "place": {
            width: 100,
            height: 100
        },
        "transition": {
            width: 30,
            height: 140
        },
        "arc": {
            strokeWidth: 5
        }
    }


    function handleMouseDown(e) {

        console.log(e.target);
        

        if(e.target.dataset.tool) {
            selectTool(e)
            return;
        }

        switch (selectedTool) {
            case "place":
            case "transition":
                if(e.target === container) {
                    addElement(e);
                    break;
                }
            case "move":
                if(e.target.dataset.draggable) {
                    dragStart(e);
                }
                break;
            case "arc":
                if(e.target.dataset.arcable) {
                    arcStart(e);
                    break;
                } else if(arcFrom && e.target === container) {
                    arcFrom = null;
                }
            case "add":
                if(e.target.dataset.arcable && items[e.target.dataset.index].type === "place") {
                    items[e.target.dataset.index].value++;
                    break;
                } else if(arcs[e.target.dataset.index]) {
                    arcs[e.target.dataset.index].value++;
                    break;
                }
            case "sub":
                if(e.target.dataset.arcable && items[e.target.dataset.index].type === "place") {
                    items[e.target.dataset.index].value--;
                    break;
                } else if(arcs[e.target.dataset.index]) {
                    arcs[e.target.dataset.index].value--;
                    break;
                }
            case "delete":
                if(e.target.dataset.arcable) {

                } else if(arcs[e.target.dataset.index]) {
                    arcs[e.target.dataset.index] = null;
                    break;
                }
            default:
                console.log('reached default');
                break;
        }
    }

    function addElement(e) {
        let id = getNextId(); 
        items.push({
            type: selectedTool,
            id: id,
            value: 0,
            name: "p"+id,
            x: e.clientX - (container.clientWidth / 2),
            y: e.clientY - (container.clientHeight / 2)
        })
    }

    function getNextId() {
        nextId++;
        return nextId;
    }

    function getNextArcId() {
        nextArcId++;
        return nextArcId;
    }

    function arcStart(e) {
        if (e.button == 0) {
            console.log('entered arcStart');
            

            if(arcFrom === null) {
                arcFrom = e.target.dataset.index;
                console.log('arcFrom set to ' + arcFrom);
                
            } else if(e.target.dataset.index === arcFrom) {
                arcFrom = null;
            }
            // arcs must go between places and transitions
            else if(e.target.dataset.index !== arcFrom 
                && items[e.target.dataset.index].type !== items[arcFrom].type
                && arcs.filter(a => (a !== null && a.from === arcFrom && a.to === e.target.dataset.index)).length === 0
            ) {
                console.log('setting arcTo');
                
                arcTo = e.target.dataset.index;
                // create an svg path between two elems
                // add arc info to state
                arcs.push({
                    id: getNextArcId(),
                    type: "arc",
                    from: arcFrom,
                    to: arcTo,
                    value: 1
                });

                arcFrom = null;
                arcTo = null;
            }
        }
    }

    function dragStart(e) {
        if (e.button == 0 && e.target !== e.currentTarget) {
            active = true;

            activeItem = e.target;

            if(activeItem !== null) {
                if(!activeItem.xOffset) {
                    activeItem.xOffset = activeItem.dataset.x;
                }

                if(!activeItem.yOffset) {
                    activeItem.yOffset = activeItem.dataset.y;
                }

                if(e.type === "touchstart") {
                    activeItem.initialX = e.touches[0].clientX - activeItem.xOffset;
                    activeItem.initialY = e.touches[0].clientY - activeItem.yOffset;
                } else {
                    activeItem.initialX = e.clientX - activeItem.xOffset;
                    activeItem.initialY = e.clientY - activeItem.yOffset;
                }
            }
        }
    }

    function dragEnd(e) {
      if(activeItem !== null) {
        activeItem.initialX = activeItem.currentX;
        activeItem.initialY = activeItem.currentY;
      }

      active = false;
      activeItem = null;
    }

    function drag(e) {
        mouseX = e.clientX - (container.clientWidth / 2);
        mouseY = e.clientY - (container.clientHeight / 2);
        if(active) {
            if(e.type === "touchmove") {
                activeItem.currentX = e.touches[0].clientX - activeItem.initialX;
                activeItem.currentY = e.touches[0].clientY - activeItem.initialY;
            } else {
                activeItem.currentX = e.clientX - activeItem.initialX;
                activeItem.currentY = e.clientY - activeItem.initialY;
            }

            activeItem.xOffset = activeItem.currentX;
            activeItem.yOffset = activeItem.currentY;

            setTranslate(activeItem.currentX, activeItem.currentY, activeItem);
        }
    }

    function setTranslate(xPos, yPos, el) {
        if(el.dataset.toolbox) {
            toolBox.x = xPos;
            toolBox.y = yPos;
        } else {
            items[el.dataset.index].x = xPos;
            items[el.dataset.index].y = yPos;
        }
    }

</script>

<style>
    #container {
      background-color: #EDEDED;
      width: 100%;
      height: 800px;
      display: flex;
      align-items: center;
      justify-content: center;
      overflow: hidden;
      border-radius: 7px;
      touch-action: none;
      border: solid black;
      position: relative;

      /* should depend on tool */
      cursor: crosshair;
    }

    .item {
      background-color:white;
      border: 10px solid #343B3F;
      touch-action: none;
      user-select: none;
      position: absolute;
      z-index: 1;

      cursor: move;
    }

    .arc {
      touch-action: none;
      user-select: none;
      position: absolute;
      z-index: 0;
    }

    .place {
        border-radius: 50%;
    }

    
    .item:hover {
      border-color: rgb(83, 111, 112);
    }

    .item:active {
      background-color: rgba(168, 218, 220, 1.00);
      border-color: rgba(168, 218, 220, 1.00);
    }

    .item-value {
        width: 1em;
        line-height: 40px;
        margin-top: 30px;
        margin-bottom: 30px;
        margin-left: 30px;
        font-size: 40px;
        text-align: center;
        pointer-events: none;
    }

    .item-label {
        text-align: center;
        font-size: 20px;
        margin-top: 35px;
        cursor: text;
    }

    .toolbox {
        height: 350px;
        background-color: darkslategray;
        position: absolute;

        z-index: 2;

        cursor: move;
        touch-action: none;
        user-select: none;

        display: flex;
        flex-direction: column;
    }

    .tool-button {
        width: 40px;
        text-align: center;
        border: solid;
        border-color: rgb(24, 44, 44);
        position: relative;
        cursor: pointer;
    }

    .tool-name {
        font-size: 20px;
        line-height: 40px;
        pointer-events: none;
        color: white;
    }

    .tool-button:hover {
        cursor: pointer;
        background-color: rgb(124, 207, 207);
    }

    input[type="radio"] {
        display: none;
    }

    .tool-selected {
        background-color: rgb(93, 158, 158);

    }

    #debugContainer {
      border-radius: 7px;
      border: solid black;
    }

    #arcsContainer {
        width: 100%;
        height: 100%;
        z-index: 0;
    }

    .item-wrapper {
        top: -25px;
        width: 150px;
        height: 150px;
        position: absolute;
        pointer-events: none;
        z-index: 0;
        left: -25px;
    }

    .arc-wrapper {
        position: absolute;
        pointer-events: none;
        z-index: 2;
    }

    .outer {
        display: none;
    }

    .item-selected {
        animation: item-selected 5s linear infinite;
        display: inherit;
        transform-origin: center;
    }

    @keyframes item-selected {
        0%   {transform: rotate(0deg)}
        50%  {transform: rotate(180deg)}
        100% {transform: rotate(360deg)}
    }

    /* tool-item hover classes */
    .move-tool.place, .move-tool.transition {
        cursor: move;
    }

    .arc-tool.place, .arc-tool.transition {
        cursor: ne-resize;
    }

    .arc-err {
        cursor: not-allowed;
    }

    .arc-label {
        font-size: 35px;
        pointer-events: all;
    }

    .arc-label:hover {
        fill: #DA4567;
    }

</style>

<div id="outerContainer">
    <div id="container" 
        bind:this={container} 
        on:touchstart|stopPropagation={dragStart}
        on:touchend|stopPropagation={dragEnd}
        on:touchmove|stopPropagation|preventDefault={drag}
        on:mousedown|stopPropagation={handleMouseDown}
        on:mouseup|stopPropagation={dragEnd}
        on:mousemove|stopPropagation={drag}
        >
        <!-- svg defs -->
        <svg width="0px" height="0px">
            <defs>
                <marker id="arrow" markerWidth="10" markerHeight="20" refX="8" refY="3" orient="auto" markerUnits="strokeWidth">
                    <path d="M0,0 L0,6 L9,3 z" fill="#f00" />
                </marker>
            </defs>
        </svg>

        <!-- toolbox -->
        <div class="toolbox" data-x={toolBox.x} data-y={toolBox.y} data-toolbox={true} data-draggable={true} style={"transform: translate3d("+toolBox.x+"px,"+toolBox.y+"px,0)"} >
            {#each tools as tool, i (tool.id)}
                <label 
                    class={"tool-button " + (tool.type == selectedTool ? "tool-selected" : "")} 
                    data-tool={true} data-id={tool.id} data-type={tool.type} 
                    for={"tool" + tool.id} style={i === 0 ? "border-bottom: none" : ""}
                >
                    <input id={"tool" + tool.id} type=radio value={tool.type}>
                    <span class="tool-name">{tool.label}</span>
                </label>
            {/each}
        </div>

        <!-- transitions and places -->
        {#each items as item, i (item.id)}
                <div 
                    class={"item " + item.type + " " + selectedTool + "-tool"} 
                    data-x={item.x} data-y={item.y} data-draggable={true} data-arcable={true} data-index={i}
                    style={"transform: translate3d("+item.x+"px,"+item.y+"px,0); width: " + sizeConfig[item.type].width + "px; height: " + sizeConfig[item.type].height + "px;"}
                     
                >
                    <svg class="item-wrapper">
                        <circle class={"outer " + (arcFrom == i ? "item-selected" : "")}
                            cx="75" cy="75" r="65" stroke="black" stroke-dasharray="5" stroke-width="2" fill="none" />
                    </svg>
                    <div style="pointer-events: none;">
                        <div class="item-value" style={item.value !== 0 ? "" : "color: transparent"}>{item.value}</div>
                        <div class="item-label">{item.name || item.id}</div>
                    </div>
                </div>
        {/each}

        <!-- arcs -->
        {#each arcs.filter(el => (el !== null)) as arc, i (arc.id)}
            <div 
                class="arc-wrapper"
                style={"transform: translate3d("+(items[arc.to].x + items[arc.from].x)/2+"px,"+(items[arc.to].y + items[arc.from].y)/2+"px,0);"} 
            >
                <svg 
                    width={Math.abs(items[arc.from].x - items[arc.to].x)} 
                    height={Math.max(Math.abs(items[arc.from].y - items[arc.to].y), sizeConfig["arc"].strokeWidth)}
                    style={"overflow: visible;"}  
                >
                    <!-- <line 
                        x1={items[arc.from].x >= items[arc.to].x ? Math.abs(items[arc.from].x - items[arc.to].x) : 0}
                        x2={items[arc.from].x < items[arc.to].x ? Math.abs(items[arc.from].x - items[arc.to].x) : 0}
                        y1={items[arc.from].y >= items[arc.to].y ? Math.abs(items[arc.from].y - items[arc.to].y) : 0} 
                        y2={items[arc.from].y < items[arc.to].y ? Math.abs(items[arc.from].y - items[arc.to].y) : 0}
                        
                    /> -->
                    <path id={"arc"+i}
                        d={"M" + (items[arc.from].x >= items[arc.to].x ? Math.abs(items[arc.from].x - items[arc.to].x) : 0)
                          + " " + (items[arc.from].y >= items[arc.to].y ? Math.abs(items[arc.from].y - items[arc.to].y) : 0)
                          + "L " + (items[arc.from].x < items[arc.to].x ? Math.abs(items[arc.from].x - items[arc.to].x) : 0)
                          + " " + (items[arc.from].y < items[arc.to].y ? Math.abs(items[arc.from].y - items[arc.to].y) : 0)}

                          stroke="black" stroke-width={sizeConfig["arc"].strokeWidth}
                        marker-end="url(#arrow)"
                    />
                    <text y="10%" class="arc-label" transform="translate(0, -10)" >
                        <textPath href={"#arc"+i} startOffset="50%" data-type="arc" data-index={i}>
                        {arc.value}
                        </textPath>
                    </text> 
                </svg>
            </div>
        {/each}
    </div>
    <div id="debugContainer">
        <p>Selected Tool: {selectedTool}</p>
        <p>X:{mouseX} Y: {mouseY} </p>
    </div>
</div>