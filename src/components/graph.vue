<template>
<div id="container" class="container1">
<div id="graph" class="graph">
  <!-- <svg></svg> -->
</div>
<div id="card" class="card1">
  <card :MaxNode='NumNode' :NumLink="NumLink" :Limit = "Limit" @childByValue="childByValue"></card>
</div>
</div>
</template>

<script type="text/javascript" src="d3.v5.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/d3-legend/2.13.0/d3-legend.js"></script>

<script>
import Card from './card.vue';
  export default {
    name: 'graph',

    data(){
      return {
        data: {},
        NumNode: Number,
        NumLink: Number,
        Limit: Number,
      }
    },
    components:{
      Card
    },
    mounted() {
      this.draw()
    },
    methods: {
      childByValue: function (childValue) {
        // childValue就是子组件传过来的值
        this.Limit = Number(childValue)
        this.draw()

      },
      clickme:function(){
        console.log(this.data)
      },

      draw: function() {
        d3.dsv(",", "board_games.csv", function(d) {
      return {
        source: d.source,
        target: d.target,
        value: +d.value
      }
    }).then(data => {

    //degree map
    let Dmap = new Map();
    data.forEach(Element => {
      if(Dmap.has(Element.source)) {
        Dmap.set(Element.source, Dmap.get(Element.source) + 1);
      } else {
        Dmap.set(Element.source, 1);
      }

      if(Dmap.has(Element.target)) {
        Dmap.set(Element.target, Dmap.get(Element.target) + 1);
      } else {
        Dmap.set(Element.target, 1);
      }
    })
    let Dmax = 0;
    Dmap.forEach(value => {
      if(value > Dmax) {
        Dmax = value;
      }
    })

    let LimitNode = Array.from(Dmap);
    this.NumNode = LimitNode.length;
    if(isNaN(this.Limit)) this.Limit = this.NumNode;

    LimitNode.sort((a, b) => a[1] > b[1] ? -1 : 1)
    LimitNode = LimitNode.slice(0, this.Limit);
    Dmap = new Map(LimitNode.map(function(i) {
      return [i[0], i[1]];
    }));


    var links = data;
    this.data = data;
    let renderNode = {}
    var renderLink = [];
  // compute the distinct nodes from the links.
  links.forEach(function(link) {
      //nodes[link.source]不存在 => nodes[link.source] = {name: link.source}
      if(Dmap.has(link.source) && Dmap.has(link.target)){
        let source = renderNode[link.source] || (renderNode[link.source] = {name: link.source});
        let target = renderNode[link.target] || (renderNode[link.target] = {name: link.target});
        renderLink.push({source, target})
      }
  });
  
  this.NumLink = renderLink.length;

  var width = 1200,
      height = 750;

  var force = d3.forceSimulation()
      .nodes(d3.values(renderNode))
      .force("link", d3.forceLink(renderLink).distance(100))
      .force('center', d3.forceCenter(width / 2, height / 2))
      .force("x", d3.forceX())
      .force("y", d3.forceY())
      .force("charge", d3.forceManyBody().strength(-250))
      .alphaTarget(1)
      .on("tick", tick);


  var parent = document.getElementById('graph');
  if(parent.children[0]){
    parent.removeChild(parent.children[0]);
  }

  var svg = d3.select("#graph")
      .append("svg")
      .attr("id","svgId")
      .attr("width", width)
      .attr("height", height);

  // add the links
  var path = svg.append("g")
      .selectAll("path")
      .data(renderLink)
      .enter()
      .append("path")
      .attr("class", function(d) { return "link" });

  // define the nodes
  var node = svg.selectAll(".node")
      .data(force.nodes())
      .enter().append("g")
      .attr("class", "node")
      .call(d3.drag()
          .on("start", dragstarted)
          .on("drag", dragged)
          .on("end", dragended))
      .on("dblclick",dbclicked)


  //linear Scale
  var linearSize = d3.scaleLinear().domain([0,Dmax]).range([5, 10]);
  var colorScale = d3.scaleQuantile().domain([0,Dmax])
            .range(["lightsalmon", "salmon", "indianred","firebrick"]);
            
  // add the nodes
  node.append("circle")
      .attr("id", function(d){
         return (d.name.replace(/\s+/g,'').toLowerCase());
      })
      .attr("r", function(d) {    
        return linearSize(Dmap.get(d.name))
      })
      .attr("fill", function(d) {
        return colorScale(Dmap.get(d.name))
      })

  //Adding node labels
  node.append("text")
      .attr("dx", 12)
      .attr("dy", ".35em")
      .text(function(d) {
          return d.name;
      });

  // add the curvy lines
  function tick() {
      path.attr("d", function(d) {
          var dx = d.target.x - d.source.x,
              dy = d.target.y - d.source.y,
              dr = Math.sqrt(dx * dx + dy * dy);
          return "M" +
              d.source.x + "," +
              d.source.y + "A" +
              dr + "," + dr + " 0 0,1 " +
              d.target.x + "," +
              d.target.y;
      });

      node.attr("transform", function(d) {
          return "translate(" + d.x + "," + d.y + ")"; 
      });
  }
  // select the svg area
  var Svg = d3.select("svg")

  //var linearSize = d3.scaleLinear().domain([0,Dmax]).range([5, 10]);

  // create a list of keys
  var keys = [Math.round(Dmax / 5,2), Math.round(Dmax / 4, 2), Math.round(Dmax / 2, 2), Dmax / 1]

  // Usually you have a color scale in your chart already
  var color = d3.scaleOrdinal()
    .domain(keys)
    .range(d3.schemeSet2);

  // Add one dot in the legend for each name.
    Svg.selectAll("mydots")
      .data(keys)
      .enter()
      .append("circle")
        .attr("cx", function(d,i){ return 100 + i*50})
        .attr("cy", 650) // 100 is where the first dot appears. 25 is the distance between dots
        .attr("r", function(d){
          return linearSize(d)
        })
        .style("fill", function(d){ 
          return colorScale(d)})

    // Add one dot in the legend for each name.
    Svg.selectAll("mylabels")
    .data(keys)
    .enter()
    .append("text")
      .attr("x", function(d,i){ return 100 + i*50 - linearSize(d)})
      .attr("y", 680) // 100 is where the first dot appears. 25 is the distance between dots
      // .style("fill", function(d){ return color(d)})
      .text(function(d){ return d})
      // .attr("text-anchor", "left")
      // .style("alignment-baseline", "middle") 
      .style("font-size","15px")         
  
         

  function dbclicked(d) {
    d3.select(this).select("circle").transition()
    .duration(1000)
    .style("fill", function(d){
      return colorScale(Dmap.get(d.name))
    });
    delete d.fx;
    delete d.fy;
    d3.select(this).classed("fixed", false);
    force.alpha(0.3).restart();
  }


  function dragstarted(d) {
    d.fixed = true;

    d3.select(this).select("circle").transition()
      .duration(0)
      .style("fill", "red")

    if (!d3.event.active) force.alphaTarget(0.3).restart();
    d.fx = d.x;
    d.fy = d.y;
  }

  function dragged(d) {
      d.fx = d3.event.x;
      d.fy = d3.event.y;
  }

  function dragended(d) {
      if (!d3.event.active) force.alphaTarget(0);
      if (d.fixed == true) {
          d.fx = d.x;
          d.fy = d.y;
      }
      else {
          d.fx = d.x;
          d.fy = d.y;
      }
  }
  
}).catch(function(error) {
  console.log(error);
});
}
}
}
</script>


<style>
.legendSize{
  color:aqua
}
  path.link {
  fill: none;
  stroke: gray;
  stroke-width: 2.5px;
}

path.link {
  fill: none;
  stroke: green;
  stroke-width: 1.5px;
  stroke-dasharray:10
}

circle {
  stroke: #fff;
  stroke: black;
  stroke-width: 1.5px;
}

text {
  fill: #000;
  font: 10px sans-serif;
  pointer-events: none;
}
.name {
  fill: #000;
  font: 20px sans-serif;
  pointer-events: none;
}
.card1{
  margin-top: 20px;
  margin-right: 20px;
  float: right;
  width: 300px;
  background: #000;
}
.graph{
  float:left;
  width: 800px;
}
.container1{
  overflow:hidden;
  /* background: grey; */
}
</style>
