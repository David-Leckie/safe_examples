<script>
  const safenetwork = require('./safenetwork_code_plus.js');
  export default {
    name: 'App',
    data() {
      return {tripText: '', trips: [],}
    },
    methods: {

      refreshList: async function() {
        this.trips = await safenetwork.getItems();
      },

      addTrip: async function() {
        const randomNumber = Math.floor((Math.random() * 10000) + 1);
        const randomKey = randomNumber.toString();
        await safenetwork.insertItem(randomKey, {text: this.tripText, made: false});
        this.tripText = '';
        await this.refreshList();
      },

      selectTrip: async function(radioTrip) {
        this.radioKey = await radioTrip.key;
        this.tripText = await radioTrip.value.text;
      },

      editTrip: async function() {
        try {
        this.selectedVersion = await safenetwork.getSelectedEntry(this.radioKey);
        await safenetwork.updateItem(this.radioKey, {text: this.tripText, made: false}, this.selectedVersion);
        let savedMessage = document.getElementById("savedMessage");
        savedMessage.className = "show";
        setTimeout(function(){ savedMessage.className = savedMessage.className.replace("show", ""); },1200);
        }
        catch (err)
        {alert ("No Trip Selected!\n\nAdd New Trip or Select From List")}
        await this.refreshList();
      },

      clearTextBox: async function()  {
        this.radioKey = '';
        this.tripText = '';
        await this.refreshList();
      },

      remaining: function() {
        var count = 0;
        this.trips.forEach((trip) => {
          count += trip.value.selected ? 0 : 1;
        });
        return count;
      },

      remove: async function() {
        let tripsToRemove = []
        await this.trips.forEach(async (trip) => {
          if (trip.value.made) tripsToRemove.push({ key: trip.key, version: trip.version });
        });

        if (tripsToRemove.length > 0) {
          await safenetwork.deleteItems(tripsToRemove);
          await this.refreshList();
        }
      }
    },

    async created() {
      await safenetwork.authoriseAndConnect();
      await safenetwork.checkForStoredMDAddress();
      await safenetwork.generateOrRetrieveMutableData();
      await safenetwork.defineCurrentMD();
      await this.refreshList();
    }
  };

  
</script>

<style scoped>
.made-true {
  text-decoration: line-through;
  color: grey;
}
#savedMessage {
  visibility: hidden;
  color: #33cc33;
}
#savedMessage.show {
  visibility: visible;
}
</style>

<template>
  <div :class="$style.App">
    <h1>Hello SAFE Network!</h1>
    <h2>Trip Planner</h2>
    <div>
      <span>{{remaining()}} of {{trips.length}} trips remaining</span>
      [ <a href="" v-on:click.prevent="remove">Delete Selected Trips</a> ]
        <form v-on:submit.prevent="editTrip">
          <ul>
            <li v-for="trip in trips">
              <input type="radio" name= "radioTrip" v-on:click="selectTrip(trip)">
              <label class="checkbox">
                <input type="checkbox" v-model="trip.value.made">
                <span v-bind:class="{ 'made-true' : trip.value.made }">{{trip.value.text}}</span><br>
              </label>
            </li>
          </ul>
            <textarea type="text" id="textBox" v-model="tripText" size="150" 
            placeholder=""></textarea>
            <input class="btn-primary" type="submit" id= "addBtn" value="Add Trip" v-on:click.prevent="addTrip">
            <input class="btn-primary" type="submit" id= "editBtn" value="Save Changes" >
            <input class="btn-primary" type="submit" id= "clearBtn" value="Clear" v-on:click.prevent="clearTextBox">
            <p></p>    
        </form>
    </div> 
<div id="savedMessage">Saved</div>
</div>
</template>

<style module>
  .App {
    padding: 20px;
  }
</style>
