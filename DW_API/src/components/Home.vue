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
        <a class="Serial" v-for="serial in serials" :key="serial.id" @click='show(serial.id,serial.title,serial.image)'>
          <div class="item" v-bind:id="serial.id">
            <img class="img-fluid" v-bind:src="serial.image" v-bind:alt="serial.title">
            <span>{{serial.season}}.{{serial.part}}: {{serial.title}}</span>
          </div>
        </a>
      </section>
      <section>
        <h1>Books</h1>
      </section>
    </div>
   <modal name="size-modal" transition="nice-modal-fade" classes="demo-modal-class" :z-index="5" :min-width="400" :min-height="200" :pivot-y="0.5" :adaptive="true" :scrollable="true" :reset="true" width="60%" height="auto">
        <div class="size-modal-content">
          <h1 id="EpisodeTitel">{{title}}</h1><div class="heart-shape"></div>
          <img class="wikifoto" alt="hier komt de (cover)foto"/>
        </div>
      </modal>
  </div>
</template>
<script>
import axios from 'axios'

export default {
  data () {
    return {
      title: 'Dit is de titel',
      serials: [],
      Doctors: []
    }
  },
  methods: {
    showDoctor () {
      console.log(event.target.value)
    },
    show (id, titel, image) {
      var self = this
      self.$modal.value.title = titel
      self.$modal.image = image
      self.$modal.show('size-modal')
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
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.wikifoto{display:inline-flex;float:right;}
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
img{max-width: 96% ;height: 10em;margin-top: 2px}
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
  /*Heart shape*/
  .heart-shape{
  position: relative;
  margin-left: 2em;
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
