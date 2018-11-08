<template>
  <div>
    <div>
      <section>
        <select name="Doctors" @change="showDoctor">
          <option id="All" key="All" value="All">All</option>
          <option v-for="Doctor in Doctors" :key="Doctor.id" :value="Doctor.id" v-bind:id="Doctor.id">{{Doctor.incarnation}}</option>
        </select>
      </section>
      <section>
        <h1>Serials</h1>
        <Spinner name="pacman" color="red" id="Spinner"/>
        <a class="Serial" v-for="serial in serials" :key="serial.id" v-on:click='show(serial.id,serial.title,serial.image)'>
          <div class="item" v-bind:id="serial.id">
            <div class="Item_Image">
              <img class="img-fluid" v-bind:src="serial.image" v-bind:alt="serial.title">
            </div>
            <div class="Item_Title">
              <span>{{serial.season}}.{{serial.part}}: {{serial.title}}</span>
            </div>
          </div>
        </a>
      </section>
      <section>
        <h1>Books</h1>
      </section>
    </div>
   <modal name="size-modal" transition="nice-modal-fade" classes="demo-modal-class" :z-index="5" :min-width="400" :min-height="200" :pivot-y="0.5" :adaptive="true" :scrollable="true" :reset="true" width="60%" height="auto">
        <div class="size-modal-content">
          <img class="wikifoto" v-bind:alt="AltCover" v-bind:src="CoverSrc"/>
          <h1 v-html="EpisodeTitel"></h1>
          <div id ="Counter_Div">
            <span id="Counter">0</span>
            <div class="heart-shape"></div>
          </div>
          <table id="Afleveringen">
            <tr>
              <td>Aflevering</td>
              <td>Runtime</td>
              <td>Airdate</td>
              <td>UK viewers</td>
            </tr>
            <tr v-for="episode in Episodes" :key="episode.id">
              <td>{{episode.title}}</td>
              <td>{{episode.runtime}}</td>
              <td>{{episode.original_air_date}}</td>
              <td>{{episode.uk_viewers_mm}}.000.000</td>
            </tr>
          </table>
        </div>
      </modal>
  </div>
</template>
<script>
import axios from 'axios'
import $ from 'jquery'
export default {
  data () {
    return {
      serials: [],
      Doctors: [],
      Episodes:[],
      EpisodeTitel:"Titel",
      AltCover:"Dit is alt",
      CoverSrc: "https://www.doctorwhofans.be/favicon.ico"
    }
  },
  methods: {
    showDoctor () {
      console.log(event.target.value)
    },
    show (id, titel, image) {
      var self = this

      self.EpisodeTitel = titel
      self.AltCover = titel
      self.CoverSrc = image
      console.log($)

      self.$modal.show('size-modal')

      axios.get('https://www.doctorwhofans.be/API/EpisodesbySerial.php?id='+id)
      .then(function GetEpisodes(res) {
        self.Episodes = res.data.Episodes
        //console.log('Data: ', res.data.Episodes)
      })
      .catch(function (error) {
        console.log('Error: ', error)
      })
    },
    hide () {
      var self = this
      self.$modal.hide('size-modal')
    }
  },
  mounted () {
    var self = this
    axios.get('https://www.doctorwhofans.be/API/Serials.php')
      .then(function getSerials (res) {
        self.serials = res.data.Serials
        // console.log('Data: ', res.data.Serials)
      })
      .catch(function (error) {
        console.log('Error: ', error)
      })
    axios.get('https://www.doctorwhofans.be/API/Doctors.php')
      .then(function GetDoctors (res) {
        self.Doctors = res.data.Doctors
        // console.log('Data: ', res.data.Doctors)
      })
      .catch(function (error) {
        console.log('Error: ', error)
      })
  }
}
$(document).ready(function () {
  setTimeout($('#Spinner').hide(), 2000)
})
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.Item_Image{    display: block;
    overflow: hidden;
    max-height: 11em;}
.Item_Title{display: block}

.wikifoto{display: inline-block;
    float: right;
    width: 38%;
    min-height: 14em;
    border: 1px solid;}
.Serial{    display: inline-block;background-color: #206090;
    width: 9em;
    height: 16em;border-radius: 5px;
    vertical-align: -webkit-baseline-middle;margin: 1.5%;color:white}
#app h1,h2 {
  font-weight: normal;
   margin-left: 2%
}
.item{text-align: center;}
a {
  color: #42b983;
}
section{margin-top: 5em;}
.Serial span{margin-top: 6%;
    display: block;}
img{width: 100% ;}
.size-modal-content {
    padding: 10px;
    font-style: 13px;
    background-color: white;
    width: 80%;
    min-height: 20em;
  }
  .v--modal-overlay[data-modal="size-modal"] {
    background: rgba(0, 0, 0, 0.5);
  }
  .demo-modal-class {
    border-radius: 5px;
    background: #F7F7F7;
    box-shadow: 5px 5px 30px 0px rgba(46,61,73, 0.6);
  }
  table{width: 100%;display: inline-block;}
  th,tr{width: 100%;}
  tr:first td{font-weight: bold}
  td{width: 41%}
  #Counter_Div{float: right;clear: both;}
  /*Heart shape*/
  .heart-shape{
  position: relative;
  display: inline-block;
  width: 10px;
  height: 10px;
  background-color: red;
}
.heart-shape:before,
.heart-shape:after{
  position: absolute;
  width: 10px;
  height: 10px;
  content: '';
  -webkit-border-radius: 50%;
  -moz-border-radius: 50%;
  -o-border-radius: 50%;
  border-radius: 50%;
  background-color: red;
}
.heart-shape:before{
  bottom: 0px;
  left: -5px;
}
.heart-shape:after{
  top: -5px;
  right: 0px;
}
.heart-shape{
  -webkit-transform: rotate(45deg);
  -moz-transform: rotate(45deg);
  -ms-transform: rotate(45deg);
  -o-transform: rotate(45deg);
  transform: rotate(45deg);
}

#Spinner{    margin: auto;
    width: 8%;}
</style>
