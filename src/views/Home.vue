<template>
  <v-app>
    <v-content>
      <Navbar />
      <v-container class="fill-height grey lighten-2" fluid>
        <v-row align="center" justify="center">
          <v-col cols="12" lg="5">
            <v-card class="elevation-12">
              <v-toolbar dark class="grey darken-3">
                <v-toolbar-title>JOIN INTERVIEW</v-toolbar-title>
              </v-toolbar>
              <v-form ref="form" class="px-3 my-3" v-model="valid">
                <v-text-field
                  label="Handle"
                  v-model="handle"
                  :counter="10"
                  prepend-icon="person"
                  :rules="handlerules"
                ></v-text-field>
                <v-text-field
                  label="Room Code"
                  v-model="room_code"
                  prepend-icon="vpn_key"
                  :rules="[v => !!v || 'Room Code is required']"
                ></v-text-field>
                <v-select
                  prepend-icon="category"
                  v-model="select"
                  :items="items"
                  :rules="[v => !!v || 'Item is required']"
                  label="Type"
                  required
                ></v-select>
                <div class="col text-center">
                  <v-btn outlined class="pink mr-4" @click="submit" :disabled="!valid">SUBMIT</v-btn>
                  <v-btn color="error" class="mr-4" @click="reset">CLEAR</v-btn>
                </div>
              </v-form>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
import axios from "axios";
import Navbar from "../components/Navbar.vue";

export default {
  components: {
    Navbar
  },
  data() {
    return {
      //rules
      host: "localhost",
      handlerules: [
        v => !!v || "Name is required",
        v => (v && v.length <= 10) || "Name must be less than 10 characters"
      ],

      handle: "",
      room_code: "",
      valid: "",
      select: "",

      items: ["CANDIDATE", "INTERVIEWER", "ADMIN"]
    };
  },

  methods: {
    submit() {
      this.$refs.form.validate();
      //console.log(this.handle,this.room_code,this.select);

      var person = {
        handle: this.handle,
        room_code: this.room_code,
        select: this.select
      };

      axios
        .get(
          `http://${this.host}:3000/api/persons/${this.handle}/${this.room_code}/${this.select}`
        )
        // eslint-disable-next-line no-unused-vars
        .then(res => {
          axios
            .post(
              `http://${this.host}:3000/api/${this.room_code}/persons/`,
              person
            )
            .then(res => console.log("Successful post", res.data))
            .catch(err => {
              console.log(err.response.data);
            });
          // io.connect('http://${this.host}:3000');

          localStorage.setItem("person", JSON.stringify(person));

          var successAlert = "";
          if (this.select == "ADMIN") {
            successAlert = "Successfully created a room.";
          } else {
            successAlert = "Successfully joined a room";
          }

          this.$swal({
            icon: "success",
            title: successAlert,
            showConfirmButton: false,
            timer: 1500
          });

          setTimeout(() => {
            this.$router.push("/join");
          }, 1600);
          //this.$router.push({name: 'Join', params: {person: person}});
        })
        .catch(err => {
          console.log(err.response.data);
          this.$swal({
            icon: "error",
            title: err.response.data,
            showConfirmButton: false,
            timer: 1500
          });
        });
    },
    reset() {
      console.log(this.$refs.form);
      this.$refs.form.reset();
    }
  },
  created() {
    console.log("home storage", JSON.parse(localStorage.getItem("person")));
    if (JSON.parse(localStorage.getItem("person") != null)) {
      this.$router.push("/join");
      return;
    }
  }
};
</script>

<style scoped>
</style>