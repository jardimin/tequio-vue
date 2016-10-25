<template>
  <div id="app" class="panel clearfix" style="padding-top: 0;">
    <div class="product-wrapper clearfix esse">
      <div class="pledge">
        <div id="graph"></div>
<!-- end progress wrapper --> 
    </div>

<!-- end pledge -->

      <div class="clearing"></div>

      <!-- <div class="rholder">
        <div class="rpdata">
          <div class="id-progress-raised"> $1,160 </div>
          <div class="id-product-funding">Pledged of $1,000 Goal</div>
        </div>
        <div class="rpdata">
          <div class="id-product-total">3</div>
          <div class="id-product-pledges">Pledgers</div>
        </div>
        <div class="rpdata">
          <div class="id-product-days">212</div>
          <div class="id-product-days-to-go">Days left</div>
        </div>
      </div>

    </div>
    <div class="id-product-proposed-end">Crowdfunding ends on
      <div class="id-widget-date">
        <div class="id-widget-month">May</div>
        <div class="id-widget-day">19</div>
        <div class="id-widget-year">2017</div>
      </div>
    </div> -->
    <div class="btn-container esse voluntario">
      <a :href="voluntarios_form" target="_blank" class="main-btn krown-button medium color">Sea Voluntário</a>
    </div>
    <div class="btn-container esse material">
      <a :href="materiais_form" target="_blank" class="main-btn krown-button medium color">Done Materiales</a>
    </div>
    <div class="btn-container esse rede">
      <a :href="rede_url" target="_blank" class="main-btn krown-button medium color">Difunda</a>
    </div>
    <div class="btn-container esse financeiro">
      <a href="" class="main-btn krown-button medium color" @click.prevent="link">Done dinero <br> (en breve)</a>
    </div>
  </div>
</template>

<script>
import _ from 'underscore'
import * as d3 from 'd3'

export default {
  name: 'app',

  data () {
    return {
      index: 0,
      rede_url: '',
      materiais_url: '',
      voluntarios_url: '',
      materiais_form: '',
      voluntarios_form: '',
      voluntarios: [],
      voluntariosOK: [],
      materiais: [],
      materiaisOK: [],
      date_start: '',
      date_end: '',
      db2: [],
      db: []
    }
  },

  watch: {
    db: function (val, oldVal) {
      if (val.length === 4) {
        this.$emit('graph-ready')
      }
    },
    index: function (val, oldVal) {
      if (val === 2) {
        let obj1 = {
          id: 'MAT',
          order: 1,
          score: (this.materiaisOK.length * 100) / this.materiais.length,
          weight: 1,
          color: '#34d293',
          label: 'Material'
        }
        let obj2 = {
          id: 'RED',
          order: 2,
          score: ((new Date().getTime() - this.date_start.getTime()) * 100) / (this.date_end.getTime() - this.date_start.getTime()),
          weight: 1,
          color: '#3ab0e2',
          label: 'Rede'
        }
        let obj3 = {
          id: 'FIN',
          order: 3,
          score: 0,
          weight: 1,
          color: '#e2cd3a',
          label: 'Financeiro'
        }
        let obj4 = {
          id: 'VOL',
          order: 4,
          score: (this.voluntariosOK.length * 100) / this.voluntarios.length,
          weight: 1,
          color: '#e96656',
          label: 'Voluntario'
        }
        this.db.push(obj3)
        this.db.push(obj4)
        this.db.push(obj1)
        this.db2.push(obj4)
        this.db2.push(obj1)
        this.db.push(obj2)
      }
    }
  },

  methods: {
    link () {
      return ''
    }
  },

  components: {
    
  },

  created: function () {
    this.$nextTick( () => {
      this.rede_url = $('#rede').text()
      this.materiais_url = $('#materiais').text()
      this.voluntarios_url = $('#voluntarios').text()
      this.materiais_form = $('#materiais_url').text()
      this.voluntarios_form = $('#voluntarios_url').text()
      this.date_start = new Date($('#data_start').text())
      this.date_end = new Date($('#data_end').text())

      let vol = this.voluntarios_url.split('/')[5]
      let mat = this.materiais_url.split('/')[5]

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
        this.index = this.index + 1
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
        this.index = this.index + 1
      })

    })
  },

  mounted: function () {
    this.$nextTick( () => {

      this.$once('graph-ready', () => {
        var width = 190,
            height = 190,
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
            .attr("class", function(d) { return d.data.id + " outlineArc"; })
            .attr("d", outlineArc);

        var path = svg.selectAll(".solidArc")
            .data(pie(this.db))
          .enter().append("path")
            .attr("fill", function(d) { return d.data.color; })
            .attr("class", function(d) { return d.data.id + " solidArc"; })
            .attr("stroke", "gray")
            .attr("d", arc)

        // calculate the weighted mean score
        var score = 
          this.db2.reduce(function(a, b) {
            //console.log('a:' + a + ', b.score: ' + b.score + ', b.weight: ' + b.weight);
            return a + (b.score * b.weight); 
          }, 0) / 
          this.db2.reduce(function(a, b) { 
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
  cursor: pointer;
}
path.outlineArc.FIN {
  fill: rgba(0,0,0,.5);
}
path.outlineArc:hover {
  fill: rgba(140,0,140,.3);
}
path.outlineArc.FIN:hover {
  fill: rgba(0,0,0,.5);
}
.product-wrapper.clearfix {
  display: none !important;
}
.product-wrapper.clearfix.esse {
  display: block !important;
}
.id-widget .id-product-proposed-end, .ignitiondeck.id-mini .id-product-proposed-end {
  display: none;
}
.btn-container {
  display: none;
}
.btn-container.esse {
  display: block;
}
.btn-container.esse.voluntario a {
  background: #e96656 !important;
}
.btn-container.esse.material a {
  background: #34d293 !important;
}
.btn-container.esse.rede a {
  background: #3ab0e2 !important;
}
.btn-container.esse.financeiro a {
  background: rgba(0,0,0,.5) !important;
}
.panel.clearfix {
  padding-top: 0 !important;
}
</style>
