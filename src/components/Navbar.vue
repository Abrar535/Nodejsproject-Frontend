<template>
  <nav>
    <v-toolbar class="grey darken-3">
      <v-btn @click="drawer = true" class="mx" fab dark color="grey">
        <v-icon dark>mdi-format-list-bulleted-square</v-icon>
      </v-btn>
      <v-toolbar-title app class="text-uppercase grey--text">
        <span class="font-weight-bold mx-3 display-1">Cefalo Interview System</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <div v-if="url == 'Join'">
        <v-btn text outlined color="purple" @click="logOut">
          <span class="font-weight-light white--text">Logout</span>
          <v-icon right>exit_to_app</v-icon>
        </v-btn>
      </div>
    </v-toolbar>

    <v-navigation-drawer v-model="drawer" absolute temporary class="grey lighten-2">
      <v-container fluid>
        <v-layout row wrap>
          <v-flex sm1 class="mx-2">
            <v-icon>people</v-icon>
          </v-flex>
          <v-flex>
            <h2>Members</h2>
            <div
              v-if="$route.name == 'Join'"
              class="subtitle-2"
            >(RoomCode:{{this.person.room_code}})</div>
          </v-flex>
        </v-layout>
        <v-divider class="black" dark></v-divider>
      </v-container>
      <v-list>
        <v-list-item v-for="link in persons" :key="link.handle">
          <v-list-item-action>
            <v-icon class="green--text">person_outline</v-icon>
            <v-icon v-if="link.handle == person.handle">done_outline</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title
              class="black--text font-weight-bold"
            >{{ link.handle + ' (' +link.select+')'}}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
  </nav>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      person: JSON.parse(localStorage.getItem("person")),
      drawer: false,
      url: null,
      persons: [],
      host: "localhost"
    };
  },
  props: ["io"],
  methods: {
    logOut() {
      if (this.person == null) return;

      var tmptext = "",
        tmpconfirmButtonText = "",
        tmptext1 = "";
      if (this.person.select != "ADMIN") {
        (tmptext = "You will leave the room!"),
          (tmpconfirmButtonText = "Yes, i want to leave!");
        tmptext1 = "You have left from the room.";
      } else {
        (tmptext = "You will delete the room!"),
          (tmpconfirmButtonText = "Yes, i want to delete this room!");
        tmptext1 = "You have successfully deleted this room.";
      }

      this.$swal({
        title: "Are you sure?",
        text: tmptext,
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: tmpconfirmButtonText
      }).then(result => {
        if (result.value) {
          localStorage.removeItem("person");
          this.io.emit("leave", this.person);
          this.$swal({
            icon: "success",
            title: "Done!",
            text: tmptext1,
            showConfirmButton: false,
            timer: 1500
          });

          axios
            .delete(
              `http://${this.host}:3000/api/${this.person.room_code}/persons/${this.person.handle}/${this.person.select}`
            )
            .then(res => {
              this.$router.push("/");

              console.log("deleted", res.data);
            })
            .catch(err => {
              console.log(err);
              this.$router.push("/");
            });
        }
      });
    }
  },
  created() {
    console.log("i am called from home");
    this.url = this.$route.name;
    if (this.url == "Home") {
      this.drawer = false;
    }
    if (this.url == "Join") {
      axios
        .get(`http://${this.host}:3000/api/${this.person.room_code}/persons/`)
        .then(res => {
          this.persons = res.data;
        })
        .catch(err => {
          console.log(err);
        });

      //console.log("testing hello hello",this.io);

      this.io.emit("newonline", this.person);
      this.io.on("newonline", data => {
        console.log("Getting new person");
        var ob = this.persons.find(p => p.handle == data.handle);
        if (!ob) {
          this.persons.push(data);
        }
      });

      this.io.on("leave", data => {
        this.persons = this.persons.filter(p => p.handle != data.handle);
      });
    }
  }
};
</script>