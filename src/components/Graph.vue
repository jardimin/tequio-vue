<template>
  <div id="graph">
    
  </div>
</template>

<script>
import * as d3 from 'd3'

export default {
  name: 'graph',
  data () {
    return {
      width: 500,
      height: 500,
      radius: Math.min(this.width, this.height) / 2,
      innerRadius: 0.3 * this.radius,
      pie: d3.pie()
          .sort(null)
          .value(function(d) { return d.width; }),
      arc: d3.arc()
          .innerRadius(this.innerRadius)
          .outerRadius(function (d) { 
            return (this.radius - this.innerRadius) * (d.data.score / 100.0) + this.innerRadius; 
          }),
      outlineArc: d3.arc()
          .innerRadius(this.innerRadius)
          .outerRadius(this.radius),
      svg: d3.select("#graph").append("svg")
          .attr("width", this.width)
          .attr("height", this.height)
          .append("g")
          .attr("transform", "translate(" + this.width / 2 + "," + this.height / 2 + "), rotate(45)")

    }
  },

  mounted: function () {
    this.$nextTick( () => {
      var self = this
      d3.csv('aster_data.csv', function(error, data) {

        data.forEach(function(d) {
          d.id     =  d.id;
          d.order  = +d.order;
          d.color  =  d.color;
          d.weight = +d.weight;
          d.score  = +d.score;
          d.width  = +d.weight;
          d.label  =  d.label;
        });
        // for (var i = 0; i < data.score; i++) { console.log(data[i].id) }
        
        var path = self.svg.selectAll(".solidArc")
            .data(self.pie(data))
          .enter().append("path")
            .attr("fill", function(d) { return d.data.color; })
            .attr("class", "solidArc")
            .attr("stroke", "gray")
            .attr("d", self.arc)

        var outerPath = self.svg.selectAll(".outlineArc")
            .data(self.pie(data))
          .enter().append("path")
            .attr("fill", "none")
            .attr("stroke", "gray")
            .attr("class", "outlineArc")
            .attr("d", self.outlineArc);  


        // calculate the weighted mean score
        var score = 
          data.reduce(function(a, b) {
            //console.log('a:' + a + ', b.score: ' + b.score + ', b.weight: ' + b.weight);
            return a + (b.score * b.weight); 
          }, 0) / 
          data.reduce(function(a, b) { 
            return a + b.weight; 
          }, 0);

        self.svg.append("svg:text")
          .attr("class", "aster-score")
          .attr("dy", ".35em")
          .attr("text-anchor", "middle") // text-align: right
          .attr("transform", "rotate(-45)")
          .text(Math.round(score));

      });
    })
  }

}
</script>

<style>
body {
  font: 10px sans-serif;
}

.axis path,
.axis line {
  fill: none;
  stroke: #000;
  shape-rendering: crispEdges;
}

.bar {
  fill: orange;
}

.solidArc:hover {
  fill: orangered ;
}

.solidArc {
    -moz-transition: all 0.3s;
    -o-transition: all 0.3s;
    -webkit-transition: all 0.3s;
    transition: all 0.3s;
}

.x.axis path {
  display: none;
}

.aster-score { 
  line-height: 1;
  font-weight: bold;
  font-size: 500%;
}

.d3-tip {
  line-height: 1;
  font-weight: bold;
  padding: 12px;
  background: rgba(0, 0, 0, 0.8);
  color: #fff;
  border-radius: 2px;
}

/* Creates a small triangle extender for the tooltip */
.d3-tip:after {
  box-sizing: border-box;
  display: inline;
  font-size: 10px;
  width: 100%;
  line-height: 1;
  color: rgba(0, 0, 0, 0.8);
  content: "\25BC";
  position: absolute;
  text-align: center;
}

/* Style northward tooltips differently */
.d3-tip.n:after {
  margin: -1px 0 0 0;
  top: 100%;
  left: 0;
}
</style>
