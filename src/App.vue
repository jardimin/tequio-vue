<template>
  <div id="app">
    <h2>Tequio</h2>
  </div>
</template>

<script>
import _ from 'underscore'
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
      materiaisOK: []
    }
  },

  components: {
    
  },

  created: function () {
    this.$nextTick( () => {
      this.rede_url = $('#rede').text()
      this.materiais_url = $('#materiais').text()
      this.voluntarios_url = $('#voluntarios').text()

      $.get(this.rede_url, function( data ) {
        data = $(data).find('.stat_value')
        console.log(data)
      });

      let vol = this.voluntarios_url.split('/')[5]
      let mat = this.materiais_url.split('/')[5]

      $.getJSON(`https://spreadsheets.google.com/feeds/list/${vol}/default/public/values?alt=json`, (data) => {
        console.log(data.feed.entry)
        for (var i = 0; i < data.feed.entry.length; i++) {
          console.log(data.feed.entry[i])
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
      })

      $.getJSON(`https://spreadsheets.google.com/feeds/list/${mat}/default/public/values?alt=json`, (data) => {
        console.log(data.feed.entry)
        for (var i = 0; i < data.feed.entry.length; i++) {
          console.log(data.feed.entry[i])
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
      })

    })
  }
}
</script>
