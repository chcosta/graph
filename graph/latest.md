<!DOCType html>
<html lang="en">
  <style>
    .axis {
      stroke: #000;
      stroke-width: 1.5px;
    }
    .node circle {
      fill: steelblue;
      stroke: steelblue;
      stroke-width: 1.5px;
    }
    .node circle.active {
        stroke: red;
        stroke-width: 3px;
    }
    .node {
      font: 10px sans-serif;
    }
    .legend {
      font: 15 px sans-serif;
      stroke: steelblue;
      stroke-width: 1.5px;
    }
    .path line {
      fill: none;
      stroke: #ccc;
      stroke-width: 0.5px;
      stroke-opacity: 0.1;
    }
    .path line.active {
      stroke: red;
      stroke-width: 1px;
      stroke-opacity: 1;
    }
  </style>

  <head>
    <meta charset="utf-8">
    <title>D3 Test</title>
      <script type="text/javascript" src="http://d3js.org/d3.v3.min.js"></script>
      <script src="https://code.jquery.com/jquery-1.10.2.js"></script>
      <script>

      // Global variables
      var repos = []; /* the unique repo names for each data node */
      var dataNodes = []; /* represents a package and its dependencies */
      var dataLinks = []; /* represents the links between packages */
      var diameter = 2000; /* diameter of circular graph */

      /* return a color based on an index */
      function getRepoColor(name) {
        var repoColor = "#fff";

        var index = repos.indexOf(name);
        if(index == "0") {
          repoColor = "#0f0";
        }
        if(index == "1") {
          repoColor = "#c37";
        }
        if(index == "2") {
          repoColor = "#ff0";
        }
        if(index == "3") {
          repoColor = "#7c3";
        }
        if (index == "4") {
            repoColor = "#0ff";
        }
        if (index == "5") {
            repoColor = "#f0f";
        }
        return repoColor;
      }

      // For each data point (node), add xy coordinates for a circular graph
      function LayoutCircularNodes(nodes, radius) {
          var angleIncrement = 360 / nodes.length;
          var currentIndex = 0;
          nodes.forEach(function (d) {
              d["radian"] = (2 * Math.PI) * (currentIndex / dataNodes.length);
              d["angle"] = d["radian"] * (180 / Math.PI);
              d["nx"] = radius * Math.cos(d["radian"]);
              d["ny"] = radius * Math.sin(d["radian"]);
              currentIndex++;
          });
      };

      // 
      function GetNodeIndex(nodes, name)
      {
          var index = 0;
          var returnIndex;
          nodes.some(function (value, index, _ary) {
              if (value.name === name) {
                  returnIndex = value.index;
                  return true;
              }
              index++;
          });
          return returnIndex;
      }

      // Update the links with endpoints defined in LayoutCircularNodes
      function UpdateDataLinks(nodes, links)
      {
          links.forEach(function (d) {
              var sourceIndex = GetNodeIndex(nodes, d.source);
              var targetIndex = GetNodeIndex(nodes, d.target);
              var x1 = 0;
              var y1 = 0;
              var x2 = 0;
              var y2 = 0;
              var t;
              if (d.source.indexOf("runtime") > -1)
              {
                  t = 0;
              }
              if (nodes[sourceIndex] !== undefined)
              {
                  x1 = nodes[sourceIndex].nx;
                  y1 = nodes[sourceIndex].ny;
              }
              if (nodes[targetIndex] !== undefined) {
                  x2 = nodes[targetIndex].nx;
                  y2 = nodes[targetIndex].ny;
              }
              d["x1"] = x1;
              d["y1"] = y1;
              d["x2"] = x2;
              d["y2"] = y2;
          });
      }

      function ParseData(data) {
          var lines = data.split(/\r\n|\n/);
          var count = 0;
          lines.forEach(function (d) {
              var tokens = d.split(",");
              var sName = tokens[0];
              if (sName !== undefined) {
                  var sRepo = tokens[1];
                  var tName = tokens[2];
                  //              var tVersion = tokens[3];

                  var cs = [];
                  lines.forEach(function (d) {
                      var ctokens = d.split(",");
                      var csName = ctokens[0];
                      var csRepo = ctokens[1];
                      var ctName = ctokens[2];
                      //                  var ctVersion = ctokens[3];
                      //                  var ctRepo = ctokens[4];
                      if (csName === sName) {
                          if (cs.indexOf(ctName) == -1) {
                              cs.push(ctName);
                          }
                      }
                  });

                  // Create each node
                  var n = {
                      name: sName,
                      children: cs,
                      repo: sRepo,
                      index: count
                  };
                  if (GetNodeIndex(dataNodes, n.name) === undefined) {
                      dataNodes.push(n);
                      count++;
                  }

                  // create data links
                  cs.forEach(function (d) {
                      var source = sName;
                      var target = d;
                      var link = {
                          source: source,
                          target: target
                      };
                      if (target !== undefined && target != "") {
                          dataLinks.push(link);
                      }
                  })
                  // Gather names of unique repos
                  if (repos.indexOf(sRepo) == -1) {
                      if (sRepo !== undefined) {
                          repos.push(sRepo);
                      }
                  }
                  //              if (repos.indexOf(tRepo) == -1) {
                  //                  if (tRepo !== undefined) {
                  //                      repos.push(tRepo);
                  //                  }
                  //              }
              }
          });
      }

      function UpdateGraphicsChart()
      {
          // Add svg to chart element
          var svg = d3.select("#chart").append("svg")
            .attr("width", diameter)
            .attr("height", diameter)
            .append("g")
            .attr("transform", "translate(" + (diameter / 2) + "," + diameter / 2 + ")");


          // Create links between nodes
          var path = svg.selectAll(".node")
            .data(dataLinks)
          .enter().append("g")
          .attr("class", "path");
          path.append("line")
              .attr("sourceName", function (d) { return d.source; })
              .attr("x1", function (d) { return d.x1; })
              .attr("x2", function (d) { return d.x2; })
              .attr("y1", function (d) { return d.y1; })
              .attr("y2", function (d) { return d.y2; });

          // Create each of the data nodes
          var node = svg.selectAll(".node")
            .data(dataNodes)
            .enter().append("g")
            .attr("class", "node");
          node.append("circle")
              .attr("r", 4.5)
              .attr("cx", function (d) { return d.nx; })
              .attr("cy", function (d) { return d.ny; })
              .attr("repo", function (d) { return d.repo; })
              .attr("name", function (d) { return d.name; })
              .on("mouseover", nodeMouseover);
          // Add text for node display
          node.append("text")
              .attr("transform", function (d) { return "translate(" + d.nx + "," + d.ny + ")rotate(" + (d.angle) + ")translate(15,0)"; })
              .text(function (d) { return d.name; });

          // Add the "external dependency" node
          svg.append("circle")
            .attr("repo", "external")
            .attr("r", 6)
            .attr("cx", 0)
            .attr("cy", 0)
            .attr("name", "external")
            .style("fill", "#f0f");
          svg.append("text")
            .text("External")
            .attr("x", "-25")
            .attr("y", "-20");

          // Update node color based on repo
          var circles = d3.selectAll("circle");
          circles.each(function (d, i) {
              var repo = d3.select(this).attr("repo");
              var rc = getRepoColor(repo);
              d3.select(this).style('fill', rc);
          });

          // Add legend for node repo colors
          var svg2 = d3.select("#legend").append("svg")
            .attr("height", 80 * repos.length)
            .append("g");
          var legends = svg2.selectAll("g")
            .data(repos)
            .enter()
            .append("g")
            .attr("class", "legend")
            .attr("transform", function (d) { return "translate(0, " + repos.indexOf(d) * 20 + ")"; });
          legends.append("rect")
            .attr("x", 0)
            .attr("y", 0)
            .attr("width", 50)
            .attr("height", 18)
            .attr("fill", function (d) {
                var rc = getRepoColor(d);
                return rc;
            });
          legends.append("text")
            .attr("dx", "4em")
            .attr("dy", "1em")
            .text(function (d) { return d; });

          // handle node mouseover
          function nodeMouseover(d) {
              svg.selectAll(".node circle").classed("active", function (p) { return p.name === d.name; });
              //d3.select(this).classed("active", true);
              svg.selectAll(".path line").classed("active", function (p) { return p.source === d.name; });
              //d3.select(this).classed("active", true);
              var dependencies = d.children;
              info.text(d.name);

              var depends = d3.select("#dependencies");
              if (dependencies !== undefined && dependencies != "")
              {
                  depends.text("Package dependencies:");
                  dependencies.forEach(function (d2) {
                      var index = GetNodeIndex(dataNodes, d2);
                      var bgcolor = "#fff";
                      var repo = "external";
                      if (dataNodes[index] !== undefined) {
                          var temprepo = dataNodes[index].repo;
                          if (temprepo != undefined)
                          {
                              repo = temprepo;
                          }
                          bgcolor = getRepoColor(repo);
                      }
                      depends.append("tr").append("td")
                        .style("background-color", bgcolor)
                        .text(d2 + " [" + repo + "]")
                  });
              }
              else
              {
                  depends.text("No package dependencies");
              }
          }
          function mouseout() {
              svg.selectAll(".active").classed("active", false);
              info.text(defaultInfo);
          }
          // initialize info text
          var info = d3.select("#package").text(defaultInfo = "Showing " + dataLinks.length + " dependencies on " + dataNodes.length + " nodes, from " + (repos.length) + " repos.");
      }

      // Main function
      function init() {
          d3.text("core-setup_coreclr_corefx_projectk-tfs_wcf.nodes2.txt",
          function (data) {
              // Parse data from txt file
              ParseData(data);
              // For each data point (node), add xy coordinates for a circular graph
              LayoutCircularNodes(dataNodes, diameter / 2 - 250);
              // Update the links with endpoints defined in LayoutCircularNodes
              UpdateDataLinks(dataNodes, dataLinks);
              // Add graphics to the web page
              UpdateGraphicsChart();
          }
        );
      };

      init();

    </script>
  </head>
<body>
    <table style="table-layout:fixed">
        <tr>
            <td width="400" id="info">
                <table width="400">
                    <tr>
                        <td id="package" style="font-weight:bold;">
                        </td>
                    </tr>
                    <tr id="dependencies">

                    </tr>
                </table>
            </td>
            <td id="chart"></td>
            <td id="legend"></td>
        </tr>

    </table>
</body>
</html>