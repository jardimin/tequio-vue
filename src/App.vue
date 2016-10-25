<template>
  <div id="app">
    <h2>Tequio</h2>
    <div id="graph"></div>

  </div>
</template>

<script>
import _ from 'underscore'
import * as d3 from 'd3'

export default {
  name: 'app',

  data () {
    return {
      rede_url: '',
      materiais_url: '',
      voluntarios_url: '',
      voluntarios: [],
      voluntariosOK: [],
      materiais: [],
      materiaisOK: [],
      db: []
    }
  },

  watch: {
    db: function (val, oldVal) {
      if (val.length === 4) {
        this.$emit('graph-ready')
      }
    }
  },

  components: {
    
  },

  created: function () {
    this.$nextTick( () => {
      this.rede_url = $('#rede').text()
      this.materiais_url = $('#materiais').text()
      this.voluntarios_url = $('#voluntarios').text()

      let vol = this.voluntarios_url.split('/')[5]
      let mat = this.materiais_url.split('/')[5]

      let obj1 = {
        id: 'AMB',
        order: 1,
        score: 58,
        weight: 1,
        color: '#34d293',
        label: 'ambiente'
      }
      let obj2 = {
        id: 'SOC',
        order: 2,
        score: 100,
        weight: 1,
        color: '#3ab0e2',
        label: 'social'
      }
      let obj3 = {
        id: 'FIN',
        order: 3,
        score: 100,
        weight: 1,
        color: '#e2cd3a',
        label: 'financeiro'
      }
      let obj4 = {
        id: 'CULT',
        order: 4,
        score: 42,
        weight: 1,
        color: '#e96656',
        label: 'cultural'
      }

      $.getJSON(`https://spreadsheets.google.com/feeds/list/${vol}/default/public/values?alt=json`, (data) => {
        for (var i = 0; i < data.feed.entry.length; i++) {
          if (data.feed.entry[i].gsx$controle.$t === 'controle') {
            let obj = {
              tipo: data.feed.entry[i].gsx$quetrabajovoluntarioestaríadispuestoarealizarpuestotarea.$t,
              nome: data.feed.entry[i].gsx$nombre.$t
            }
            this.voluntarios.push(obj)
          } else if (data.feed.entry[i].gsx$controle.$t == 'ok') {
            let obj = {
              tipo: data.feed.entry[i].gsx$quetrabajovoluntarioestaríadispuestoarealizarpuestotarea.$t,
              nome: data.feed.entry[i].gsx$nombre.$t
            }
            this.voluntariosOK.push(obj)
          }
        };
        this.db.push(obj3)
        this.db.push(obj4)
      })

      $.getJSON(`https://spreadsheets.google.com/feeds/list/${mat}/default/public/values?alt=json`, (data) => {
        for (var i = 0; i < data.feed.entry.length; i++) {
          if (data.feed.entry[i].gsx$controle.$t === 'controle') {
            let obj = {
              tipo: data.feed.entry[i].gsx$puedecolaborardonandoalgunodeestosmaterialesmaterialfin.$t,
              nome: data.feed.entry[i].gsx$nombre.$t
            }
            this.materiais.push(obj)
          } else if (data.feed.entry[i].gsx$controle.$t == 'ok') {
            let obj = {
              tipo: data.feed.entry[i].gsx$puedecolaborardonandoalgunodeestosmaterialesmaterialfin.$t,
              nome: data.feed.entry[i].gsx$nombre.$t
            }
            this.materiaisOK.push(obj)
          }
        };
        this.db.push(obj1)
        this.db.push(obj2)
      })

    })
  },

  mounted: function () {
    this.$nextTick( () => {

      this.$once('graph-ready', () => {
        var width = 500,
            height = 500,
            radius = Math.min(width, height) / 2,
            innerRadius = 0.3 * radius;

        var pie = d3.pie()
            .sort(null)
            .value(function(d) { return d.width; });

        var arc = d3.arc()
          .innerRadius(innerRadius)
          .outerRadius(function (d) { 
            return (radius - innerRadius) * (d.data.score / 100.0) + innerRadius; 
          });

        var outlineArc = d3.arc()
                .innerRadius(innerRadius)
                .outerRadius(radius);

        var svg = d3.select("#graph").append("svg")
            .attr("width", width)
            .attr("height", height)
            .append("g")
            .attr("transform", "translate(" + width / 2 + "," + height / 2 + "), rotate(45)")


        this.db.forEach(function(d) {
          d.id     =  d.id;
          d.order  = +d.order;
          d.color  =  d.color;
          d.weight = +d.weight;
          d.score  = +d.score;
          d.width  = +d.weight;
          d.label  =  d.label;
        });
        // for (var i = 0; i < data.score; i++) { console.log(data[i].id) }
        
        var outerPath = svg.selectAll(".outlineArc")
            .data(pie(this.db))
          .enter().append("path")
            .attr("fill", "none")
            .attr("stroke", "gray")
            .attr("class", "outlineArc")
            .attr("d", outlineArc);  

        var path = svg.selectAll(".solidArc")
            .data(pie(this.db))
          .enter().append("path")
            .attr("fill", function(d) { return d.data.color; })
            .attr("class", "solidArc")
            .attr("stroke", "gray")
            .attr("d", arc)

        // calculate the weighted mean score
        var score = 
          this.db.reduce(function(a, b) {
            //console.log('a:' + a + ', b.score: ' + b.score + ', b.weight: ' + b.weight);
            return a + (b.score * b.weight); 
          }, 0) / 
          this.db.reduce(function(a, b) { 
            return a + b.weight; 
          }, 0);

        svg.append("svg:text")
          .attr("class", "aster-score")
          .attr("dy", ".35em")
          .attr("text-anchor", "middle") // text-align: right
          .attr("transform", "rotate(-45)")
          .text(Math.round(score) + "%");
      })


    })
  }
}
</script>
<style>
path.solidArc {
  stroke: white;
  cursor: pointer;
}
path.solidArc:hover {
  opacity: .8;
}
path.outlineArc {
  stroke: white;
  fill: rgba(0,0,0,.1);
}
</style>
