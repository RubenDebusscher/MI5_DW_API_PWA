<template @detected-condition="handleConnectivityChange">
  <div>
    <div>
      <article id='first_Article'>
        <offline @detected-condition="handleConnectivityChange">
    <!-- Only renders when the device is online -->
    <!-- Only renders when the device is offline -->
    <div slot="offline">
      <p>You appear to be offline, that's okay, we can still do things...  The episode data will not be up to date though.</p>
    </div>
  </offline>
        <h1>Serials</h1>
        <section>
          <section>
            <label for="Doctors">Choose your Doctor:</label>
            <select id="Doctors" name="Doctors" v-on:change="showDoctor">
              <option id="All" key="All" value="All">All</option>
              <option v-for="Doctor in Doctors" :key="Doctor.id" :value="Doctor.id" v-bind:id="Doctor.incarnation">{{Doctor.incarnation}}</option>
            </select>
          </section>
          <Spinner name="pacman" color="red" id="Spinner"/>
          <a class="Serial" v-for="serial in serials" :key="serial.id"
          v-on:click='show(serial.id,serial.title,serial.image,serial.external_link)'>
            <div class="item" v-bind:id="serial.id">
              <div class="Item_Image">
                <!-- pas hier een extra if toe om een default te voorzien als de staus offline is-->
                <img v-if="serial.image !=''" class="img-fluid" v-bind:src="serial.image" v-bind:alt="serial.title">
                <img v-else class="img-fluid" src="static/img/favicon.ico" v-bind:alt="serial.title">
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
   <modal name="size-modal" transition="nice-modal-fade" classes="demo-modal-class"
   :z-index="5" :min-width="400" :min-height="200" :pivot-y="0.5" :adaptive="true"
   :scrollable="true" :reset="true" width="60%" height="auto">
        <div class="size-modal-content">
          <button v-on:click="CloseModal()">Close</button>
          <img v-if="CoverSrc !=''" class="wikifoto" v-bind:alt="AltCover"
          v-bind:src="CoverSrc"/>
          <img v-else  class="wikifoto" v-bind:alt="AltCover"
          src="static/img/favicon.ico"/>
          <h1><a v-bind:href="Link" v-html="EpisodeTitel" target="_blank"></a></h1>
          <h2>Doctor(s)</h2>
          <ul>
            <li v-for="Doctor in DoctorsforEpisode" :key="Doctor.doctor_id">
              {{Doctor.incarnation}}
            </li>
          </ul>
          <h2>Companions</h2>
          <ul>
            <li v-for="Companion in CompanionsforEpisode"
            :key="Companion.companion_id">
              {{Companion.personage}}
            </li>
          </ul>
          <div id ="Counter_Div">
            <span id="Counter">0</span>
            <div class="heart-shape" v-on:click="Heartclicked()"></div>
          </div>
          <!--voeg hier een if toe voor het geval offline, toon deze data dan niet maar een boodschap-->
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
import offline from 'v-offline'
export default {
  components: {
    offline
  },
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
      CoverSrc: 'static/img/favicon.ico',
      Link: '/'
    }
  },
  methods: {
    handleConnectivityChange (status) {
      var allImages = document.getElementsByTagName('img')
      for (var i = 0; i < allImages.length; i++) {
        if (!allImages[i].complete) {
          allImages[i].src = 'static/img/favicon.ico'
        }
      }
      console.log(status)
    },
    showDoctor () {
      var self = this
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
            self.serials = res.data.Serials
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
    },
    CloseModal () {
      var self = this
      self.$modal.hide('size-modal')
    },
    Heartclicked () {
      // insert click in database
      // re-get the amount of hearts for selected episode
      var aantal = parseInt($('#Counter').text(), 10)
      $('#Counter').text(aantal += 1)
    }
  },
  watch: {
    serials: {
      handler () {
        console.log('Serials changed!')
        localStorage.setItem('serials', JSON.stringify(this.serials))
      },
      deep: true
    },
    Doctors: {
      handler () {
        console.log('Doctors changed!')
        localStorage.setItem('Doctors', JSON.stringify(this.Doctors))
      },
      deep: true
    },
    Episodes: {
      handler () {
        console.log('Episodes changed!')
        localStorage.setItem('Episodes', JSON.stringify(this.Episodes))
      },
      deep: true
    },
    offline
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
    console.log('App mounted!')
    if (localStorage.getItem('serials')) this.serials = JSON.parse(localStorage.getItem('serials'))
    if (localStorage.getItem('Doctors')) this.Doctors = JSON.parse(localStorage.getItem('Doctors'))
    if (localStorage.getItem('Episodes')) this.Episodes = JSON.parse(localStorage.getItem('Episodes'))
  }
}
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
button {
  background-color: #f44336;; /* Green */
  border: none;
  color: white;
  padding: 15px 32px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
}
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
  #Counter_Div{float: right;clear: both;margin-right: 1em;}
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
tr:nth-child(even){background-color: wheat;}
tr:nth-child(odd){background-color: white;}
td{border-collapse: collapse}
#Spinner{    margin: auto;
    width: 8%;}
@media only screen and (max-width: 660px) {
  .Serial{width: 6em;
    height: 13em;}
}
</style>
