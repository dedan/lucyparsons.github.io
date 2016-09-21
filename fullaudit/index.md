---
layout: page
title: The Narcotics Audit
header:
    image_fullwidth: "RekiaBoyd.jpg"
---

## Five Years and Counting...

Since 2015, Lucy Parsons Labs has obtained approximately six hundred checks related to the Chicago Police Department's use of civil asset forfeiture. This page contains a searchable and sortable repository of these records, as well as all the checks we've obtained since 2009. There are links to the Freedom of Information Act (FOIA) requests, the results of each request, and the check amounts. We also invite you to read our full investigation into CPD's use of civil asset forfeiture in the [Chicago Reader](link).

Our investigation found that the majority of the civil asset forfeiture funds (known as the "1505" fund) went to pay for the everyday prosecution of the War on Drugs. Alarmingly, the 1505 fund has been used to purchase a large portion of the Chicago Police Department's surveillance equipment. This includes controversial equipment such as IMSI catchers (commonly known as Stingrays), Automatic License Plate Readers, "blue light cameras" (or PODs), and PenLINK equipment. You can find surveillance equipment by sorting the categories tab. Lucy Parsons Labs had created a overview of the surveillance equipment that Chicago Police Department [uses in Chicago](https://redshiftzero.github.io/policesurveillance) which contains most of the tools in this chart. 

The Chicago Police Department collected between $4.7 million and $9.3 million annually from 2009 to 2015. Between January 2010 and March 2016, $16 million, or about 80 percent of these larger payments, was spent on the day-to-day operations of the Bureau of Organized Crime. In 2015, the Bureau of Organized Crime received $77 million in its official budget. That year's additional forfeiture income was $4.7 million, nearly 6 percent of its total budget. Yet the forfeiture fund was already flush with cash from previous years: at the end of October 2015, the bureau had more than $17 million in its forfeiture checking and savings accounts. A PDF of an [internal CPD audit](https://github.com/lucyparsons/1505analysis/blob/master/MARTINEZv.CPD-RESPONSIVE%20RECORDS1505AUDIT.pdf) obtained by Lucy Parsons Labs breaks down some of BoC expenses by category. In 2010, $267,000 was used for equipment and tools, $28,000 for office supplies, and almost $1 million for telephonic equipment and services.

*Note:* There are unknowns in the data set due to liberal use of redactions by the Chicago Police's FOIA Department as well as unfulfilled requests. There are also expenditures that point to spefic companies but we don't have enough context to definitively state the purpose of the purchase. These are both classified under the _Unknown_ category. If you would like to participate in this project, please view our project page on [MuckRock](https://www.muckrock.com/project/opening-the-chicago-surveillance-fund-25/).

{::options parse_block_html="true" /}

<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!-- The above 3 meta tags *must* come first in the head; any other head content must come *after* these tags -->

    <!-- HTML5 shim and Respond.js for IE8 support of HTML5 elements and media queries -->
    <!--[if lt IE 9]>
      <link href="css/bootstrap.min.css" rel="stylesheet">
      <link href="css/dataTables.bootstrap.css" rel="stylesheet">
      <script src="js/html5shiv.min.js"></script>
      <script src="js/respond.min.js"></script>
    <![endif]-->
  </head>
<body>
 <div class="d3svg">
<style>
circle,
path {
  cursor: pointer;
}

circle {
  fill: none;
  pointer-events: all;
}

#tooltip { background-color: white;
                          padding: 3px 5px;
                          border: 1px solid black;
                          text-align: center;}
html {
        font-family: sans-serif;
}
</style>

<script src="js/d3.v3.min.js"></script>
<script>
var margin = {top: 300, right: 380, bottom: 300, left: 360},
    radius = Math.min(margin.top, margin.right, margin.bottom, margin.left) - 10;

function filter_min_arc_size_text(d, i) {return (d.dx*d.depth*radius/3)>14}; 

var hue = d3.scale.category20();

var luminance = d3.scale.sqrt()
    .domain([0, 1e6])
    .clamp(true)
    .range([90, 20]);

var svg = d3.select("body").append("svg")
    .attr("width", margin.left + margin.right)
    .attr("height", margin.top + margin.bottom)
  .append("g")
    .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

var partition = d3.layout.partition()
    .sort(function(a, b) { return d3.ascending(a.name, b.name); })
    .size([2 * Math.PI, radius]);

var arc = d3.svg.arc()
    .startAngle(function(d) { return d.x; })
    .endAngle(function(d) { return d.x + d.dx - .01 / (d.depth + .5); })
    .innerRadius(function(d) { return radius / 3 * d.depth; })
    .outerRadius(function(d) { return radius / 3 * (d.depth + 1) - 1; });

//Tooltip description
var tooltip = d3.select("body")
    .append("div")
    .attr("id", "tooltip")
    .style("position", "absolute")
    .style("z-index", "10")
    .style("opacity", 0);

function format_number(x) {
  return x.toFixed(2).toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
}


function format_description(d) {
  var description = d.description;
      return  '<b>' + d.name + '</b><br> ($' + format_number(d.value) + ')';
}

function computeTextRotation(d) {
        var angle=(d.x +d.dx/2)*180/Math.PI - 90    
        
        return angle;
}

function mouseOverArc(d) {
         d3.select(this).attr("stroke","black")
          tooltip.html(format_description(d));
          return tooltip.transition()
            .duration(50)
            .style("opacity", 0.9);
        }

function mouseOutArc(){
        d3.select(this).attr("stroke","")
        return tooltip.style("opacity", 0);
}

function mouseMoveArc (d) {
          return tooltip
            .style("top", (d3.event.pageY-10)+"px")
            .style("left", (d3.event.pageX+10)+"px");
}

var root_ = null;
d3.json("output.json", function(error, root) {
  if (error) return console.warn(error);
  // Compute the initial layout on the entire tree to sum sizes.
  // Also compute the full name and fill color for each node,
  // and stash the children so they can be restored as we descend.
        
  partition
      .value(function(d) { return d.size; })
      .nodes(root)
      .forEach(function(d) {
        d._children = d.children;
        d.sum = d.value;
        d.key = key(d);
        d.fill = fill(d);
      });

  // Now redefine the value function to use the previously-computed sum.
  partition
      .children(function(d, depth) { return depth < 2 ? d._children : null; })
      .value(function(d) { return d.sum; });

  var center = svg.append("circle")
      .attr("r", radius / 3)
      .on("click", zoomOut);

  center.append("title")
      .text("zoom out");
      
  var partitioned_data=partition.nodes(root).slice(1)

  var path = svg.selectAll("path")
      .data(partitioned_data)
    .enter().append("path")
      .attr("d", arc)
      .style("fill", function(d) { return d.fill; })
      .each(function(d) { this._current = updateArc(d); })
      .on("click", zoomIn)
                .on("mouseover", mouseOverArc)
      .on("mousemove", mouseMoveArc)
      .on("mouseout", mouseOutArc);
  
      
  var texts = svg.selectAll("text")
      .data(partitioned_data)
    .enter().append("text")
                .filter(filter_min_arc_size_text)       
        .attr("transform", function(d) { return "rotate(" + computeTextRotation(d) + ")"; })
                .attr("x", function(d) { return radius / 3 * d.depth; })    
                .attr("dx", "6") // margin
      .attr("dy", ".35em") // vertical-align    
                .text(function(d,i) {return d.name})

  function zoomIn(p) {
    if (p.depth > 1) p = p.parent;
    if (!p.children) return;
    zoom(p, p);
  }

  function zoomOut(p) {
    if (!p.parent) return;
    zoom(p.parent, p);
  }

  // Zoom to the specified new root.
  function zoom(root, p) {
    if (document.documentElement.__transition__) return;

    // Rescale outside angles to match the new layout.
    var enterArc,
        exitArc,
        outsideAngle = d3.scale.linear().domain([0, 2 * Math.PI]);

    function insideArc(d) {
      return p.key > d.key
          ? {depth: d.depth - 1, x: 0, dx: 0} : p.key < d.key
          ? {depth: d.depth - 1, x: 2 * Math.PI, dx: 0}
          : {depth: 0, x: 0, dx: 2 * Math.PI};
    }

    function outsideArc(d) {
      return {depth: d.depth + 1, x: outsideAngle(d.x), dx: outsideAngle(d.x + d.dx) - outsideAngle(d.x)};
    }

    center.datum(root);

    // When zooming in, arcs enter from the outside and exit to the inside.
    // Entering outside arcs start from the old layout.
    if (root === p) enterArc = outsideArc, exitArc = insideArc, outsideAngle.range([p.x, p.x + p.dx]);
         var new_data=partition.nodes(root).slice(1)
    path = path.data(new_data, function(d) { return d.key; });
                 
         // When zooming out, arcs enter from the inside and exit to the outside.
    // Exiting outside arcs transition to the new layout.
    if (root !== p) enterArc = insideArc, exitArc = outsideArc, outsideAngle.range([p.x, p.x + p.dx]);

    d3.transition().duration(d3.event.altKey ? 7500 : 750).each(function() {
      path.exit().transition()
          .style("fill-opacity", function(d) { return d.depth === 1 + (root === p) ? 1 : 0; })
          .attrTween("d", function(d) { return arcTween.call(this, exitArc(d)); })
          .remove();
          
      path.enter().append("path")
          .style("fill-opacity", function(d) { return d.depth === 2 - (root === p) ? 1 : 0; })
          .style("fill", function(d) { return d.fill; })
          .on("click", zoomIn)
                         .on("mouseover", mouseOverArc)
         .on("mousemove", mouseMoveArc)
         .on("mouseout", mouseOutArc)
          .each(function(d) { this._current = enterArc(d); });

      path.transition()
          .style("fill-opacity", 1)
          .attrTween("d", function(d) { return arcTween.call(this, updateArc(d)); });
    });
         texts = texts.data(new_data, function(d) { return d.key; })
         texts.exit()
                 .remove()    
    texts.enter()
            .append("text")
        
    texts.style("opacity", 0)
      .attr("transform", function(d) { return "rotate(" + computeTextRotation(d) + ")"; })
                .attr("x", function(d) { return radius / 3 * d.depth; })    
                .attr("dx", "6") // margin
      .attr("dy", ".35em") // vertical-align
      .filter(filter_min_arc_size_text)     
      .text(function(d,i) {return d.name})
                .transition().delay(750).style("opacity", 1)
        
                 
  }
});

function key(d) {
  var k = [], p = d;
  while (p.depth) k.push(p.name), p = p.parent;
  return k.reverse().join(".");
}

function fill(d) {
  var p = d;
  while (p.depth > 1) p = p.parent;
  var c = d3.lab(hue(p.name));
  c.l = luminance(d.sum);
  return c;
}

function arcTween(b) {
  var i = d3.interpolate(this._current, b);
  this._current = i(0);
  return function(t) {
    return arc(i(t));
  };
}

function updateArc(d) {
  return {depth: d.depth, x: d.x, dx: d.dx};
}

d3.select(self.frameElement).style("height", margin.top + margin.bottom + "px");

</script> 
</div>

    <div id='table-container'></div>
    
<!-- Bootstrap core JavaScript
    ================================================== -->
    <!-- Placed at the end of the document so the pages load faster -->
    <script src="js/jquery.min.js"></script>
    <script src="js/bootstrap.min.js"></script>
    <script src="js/jquery.dataTables.min.js"></script>
    <script src='js/csv_to_html_table.js'></script>
    <script type="text/javascript" src="js/jquery.csv.min.js"></script>
    <script>
      init_table({
        csv_path: 'data/August24clean.csv',
        element: 'table-container', 
        allow_download: true,
        csv_options: {separator: ',', delimiter: '"'},
        datatables_options: {"paging": true}
      });
    </script>
    </div>
</body>
</html>