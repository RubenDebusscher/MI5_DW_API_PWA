<template>
  <div>
    <div>
      <article id='first_Article'>
        <h1>Serials</h1>
        <section>
          <section>
            <label for="Doctors">Choose your Doctor:</label>
            <select id="Doctors" name="Doctors" v-on:change="showDoctor" v-model="Episodes">
              <option id="All" key="All" value="All">All</option>
              <option v-for="Doctor in Doctors" :key="Doctor.id" :value="Doctor.id" v-bind:id="Doctor.incarnation">{{Doctor.incarnation}}</option>
            </select>
          </section>
          <Spinner name="pacman" color="red" id="Spinner"/>
          <a class="Serial" v-for="serial in serials" :key="serial.id" v-on:click='show(serial.id,serial.title,serial.image,serial.external_link)'>
            <div class="item" v-bind:id="serial.id">
              <div class="Item_Image">
                <img v-if="serial.image !=''" class="img-fluid" v-bind:src="serial.image" v-bind:alt="serial.title">
                <img v-else class="img-fluid" src="https://www.doctorwhofans.be/favicon.ico" v-bind:alt="serial.title">
              </div>
              <div class="Item_Title">
                <span>{{serial.season}}.{{serial.part}}: {{serial.title}}</span>
              </div>
            </div>
          </a>
        </section>
      </article>
      <article>
        <h1>Books</h1>
      </article>
      <article>
        <h1>Audio</h1>
      </article>
      <article>
        <h1>Magazines</h1>
      </article>
    </div>
   <modal name="size-modal" transition="nice-modal-fade" classes="demo-modal-class" :z-index="5" :min-width="400" :min-height="200" :pivot-y="0.5" :adaptive="true" :scrollable="true" :reset="true" width="60%" height="auto">
        <div class="size-modal-content">
          <img v-if="CoverSrc !=''" class="wikifoto" v-bind:alt="AltCover" v-bind:src="CoverSrc"/>
          <img v-else  class="wikifoto" v-bind:alt="AltCover" src="https://www.doctorwhofans.be/favicon.ico"/>
          <h1><a v-bind:href="Link" v-html="EpisodeTitel" target="_blank"></a></h1>
          <h2>Doctor(s)</h2>
          <ul>
            <li v-for="Doctor in DoctorsforEpisode" :key="Doctor.doctor_id">{{Doctor.incarnation}}</li>
          </ul>
          <h2>Companions</h2>
          <ul>
            <li v-for="Companion in CompanionsforEpisode" :key="Companion.companion_id">{{Companion.personage}}</li>
          </ul>
          <div id ="Counter_Div">
            <span id="Counter">0</span>
            <div class="heart-shape" onclick="Heartclicked"></div>
          </div>
          <table id="Afleveringen">
            <tr>
              <th>Aflevering</th>
              <th>Runtime</th>
              <th>Airdate</th>
              <th>UK viewers</th>
              <th>AI</th>
            </tr>
            <tr v-for="episode in Episodes" :key="episode.id">
              <td v-if="episode.missing==1&& episode.recreated==0" style="background-color:red;color:white">{{episode.title}}</td>
              <td v-else-if="episode.recreated==1&& episode.missing==1" style="background-color:red;color:white">{{episode.title}} (recreated)</td>
              <td v-else>{{episode.title}}</td>
              <td>{{episode.runtime}}</td>
              <td>{{episode.original_air_date}}</td>
              <td>{{episode.uk_viewers_mm}}</td>
              <td>{{episode.appreciation_index}}</td>
            </tr>
          </table>
          <h2>Synopsis</h2>
          <article v-for="episode in Episodes" :key="episode.id" class="small">
            <h3>{{episode.title}}</h3>
            <span>{{episode.synopsis}}</span>
          </article>
        </div>
      </modal>
  </div>
</template>
<script>
import axios from 'axios'
import $ from 'jquery'
import Vue from 'vue'
export default {
  data () {
    return {
      serials: [],
      Doctors: [],
      Episodes: [],
      selected: 'All',
      CompanionsforEpisode: [],
      DoctorsforEpisode: [],
      EpisodeTitel: 'Titel',
      AltCover: 'Dit is alt',
      CoverSrc: 'https://www.doctorwhofans.be/favicon.ico',
      Link: '/'
    }
  },
  methods: {
    showDoctor () {
      if (event.target.value === 'All') {
        axios.get('https://www.doctorwhofans.be/API/Serials.php')
          .then(function getSerials (res) {
            self.serials = res.data.Serials
          })
          .catch(function (error) {
            console.log('Error: ', error)
          })
      } else {
        axios.get('https://www.doctorwhofans.be/API/SerialsbyDoctor.php?id=' + event.target.value)
          .then(function getSerials (res) {
            // self.serials = res.data.Serials
            for (var i = self.serials.length - 1; i >= 0; i--) {
              Vue.delete(self.serials, i)
            }
            for (i = 0; i < res.data.Serials.length; i++) {
              Vue.$set(self.serials, i, res.data.Serials[i])
            }
            console.log(self.serials)
          })
          .catch(function (error) {
            console.log('Error: ', error)
          })
      }
    },
    show (id, titel, image, link) {
      var self = this
      self.EpisodeTitel = titel
      self.AltCover = titel
      self.CoverSrc = image
      self.Link = link
      self.$modal.show('size-modal')

      axios.get('https://www.doctorwhofans.be/API/EpisodesbySerial.php?id=' + id)
        .then(function GetEpisodes (res) {
          self.Episodes = res.data.Episodes
        })
        .catch(function (error) {
          console.log('Error: ', error)
        })
      axios.get('https://www.doctorwhofans.be/API/CompanionsforEpisode.php?id=' + id)
        .then(function GetCompanions (res) {
          self.CompanionsforEpisode = res.data.Companions
        })
        .catch(function (error) {
          console.log('Error: ', error)
        })
      axios.get('https://www.doctorwhofans.be/API/DoctorsforEpisode.php?id=' + id)
        .then(function GetDoctorsforEpisode (res) {
          self.DoctorsforEpisode = res.data.DoctorsforEpisode
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
  watch () {
    var self = this
    // serials: self.serials
    return self.serials
  },
  mounted () {
    var self = this
    axios.get('https://www.doctorwhofans.be/API/Serials.php')
      .then(function getSerials (res) {
        self.serials = res.data.Serials
        $('#Spinner').hide()
      })
      .catch(function (error) {
        console.log('Error: ', error)
      })
    axios.get('https://www.doctorwhofans.be/API/Doctors.php')
      .then(function GetDoctors (res) {
        self.Doctors = res.data.Doctors
      })
      .catch(function (error) {
        console.log('Error: ', error)
      })
  }
}
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.Item_Image{    display: block;
    overflow: hidden;
    max-height: 11em;}
.Item_Title{display: block}

.wikifoto{display: block;
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
#first_Article{padding-top: 5em;}
.small{margin-top: 0em}
.Serial span{margin-top: 6%;
    display: block;}
img{width: 100% ;}
.size-modal-content {
    padding: 10px;
    font-style: 13px;
    background-color: white;
    width: 98%;
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
  th{max-width: 100%;
    width: 29%;
    min-width: auto;
    text-align: left;
}
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
