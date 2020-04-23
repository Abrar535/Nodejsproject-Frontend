

<template>
  <v-app>
    <v-content>
      <Navbar :io="this.io" />
      <v-container fluid class="fill-height grey lighten-2">
        <v-container>
          <v-alert prominent type="warning" v-if="person.select == 'CANDIDATE' && submitalert">
            <v-row align="center">
              <v-col
                class="grow"
              >Your solution has been submitted to the interviewers and cannot be edited by you anymore. Please wait for a verdict</v-col>
              <v-col class="shrink">
                <v-btn @click="submitalert = false">Close</v-btn>
              </v-col>
            </v-row>
          </v-alert>

          <v-alert prominent type="success" v-if="person.select != 'CANDIDATE' && submitalert">
            <v-row align="center">
              <v-col class="grow">The Candidate has successfully submitted his solution.</v-col>
              <v-col class="shrink">
                <v-btn @click="submitalert = false">Close</v-btn>
              </v-col>
            </v-row>
          </v-alert>

          <v-alert prominent type="success" v-if="person.select != 'CANDIDATE' && acceptalert">
            <v-row align="center">
              <v-col class="grow">The Candidate's solution has been accepted by an Interviewer</v-col>
              <v-col class="shrink">
                <v-btn @click="acceptalert = false">Close</v-btn>
              </v-col>
            </v-row>
          </v-alert>

          <v-alert prominent type="success" v-if="person.select == 'CANDIDATE' && acceptalert">
            <v-row align="center">
              <v-col class="grow">Congratulations! An interviewer has accepted your solution.</v-col>
              <v-col class="shrink">
                <v-btn @click="acceptalert = false">Close</v-btn>
              </v-col>
            </v-row>
          </v-alert>

          <v-alert prominent type="error" v-if="person.select == 'CANDIDATE' && rejectalert">
            <v-row align="center">
              <v-col class="grow">Unfortunately your solution has been rejected by an Interviewer.</v-col>
              <v-col class="shrink">
                <v-btn @click="rejectalert = false">Close</v-btn>
              </v-col>
            </v-row>
          </v-alert>

          <v-alert prominent type="error" v-if="person.select != 'CANDIDATE' && rejectalert">
            <v-row align="center">
              <v-col class="grow">An Interviewer has rejected the solution of the candidate.</v-col>
              <v-col class="shrink">
                <v-btn @click="rejectalert = false">Close</v-btn>
              </v-col>
            </v-row>
          </v-alert>
        </v-container>

        <v-layout row wrap justify-end>
          <v-flex md9>
            <div style="width:1400px ;transform:translate(-50px);">
              <v-expansion-panels
                multiple
                :value="panel_array"
                ref="panel"
                popout
                hover
                class="px-12 my-3 panelalign"
              >
                <!-- Question -->

                <v-expansion-panel class="ma-2">
                  <v-expansion-panel-header class="blue darken-2 font-weight-bold title">Question</v-expansion-panel-header>
                  <v-expansion-panel-content>
                    {{question}}
                    <div class="text-right">
                      <v-dialog v-model="dialog" width="500">
                        <template v-if="person.select != 'CANDIDATE'" v-slot:activator="{ on }">
                          <v-btn
                            text
                            class="success darken-2 my-1"
                            v-on="on"
                            @click="tmpquestion = question"
                          >
                            <div v-if="question == ''">
                              <v-icon>add</v-icon>
                              <span class="font-weight-light white--text">Add</span>
                            </div>
                            <div v-else>
                              <v-icon>create</v-icon>
                              <span class="font-weight-light white--text">Edit</span>
                            </div>
                          </v-btn>
                        </template>

                        <v-card style>
                          <v-card-title class="headline grey lighten-2" primary-title>Add Question</v-card-title>

                          <v-textarea v-model="tmpquestion" background-color="grey lighten-3"></v-textarea>

                          <v-divider></v-divider>

                          <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn color="primary" text @click="questionConfirm">Confirm</v-btn>
                            <v-btn color="primary" text @click="questionCancel">Cancel</v-btn>
                          </v-card-actions>
                        </v-card>
                      </v-dialog>
                    </div>
                  </v-expansion-panel-content>
                </v-expansion-panel>
                <!-- Codemirror -->

                <v-expansion-panel style="transform: translate(10px);">
                  <v-expansion-panel-header class="blue darken-2 font-weight-bold title">Code</v-expansion-panel-header>
                  <v-expansion-panel-content>
                    <div>
                      <codemirror
                        ref="myCm"
                        v-model="code"
                        :options="cmOptions"
                        @focus="onCmFocus"
                      />
                    </div>

                    <div class="text-right">
                      <v-checkbox v-model="inputcheckbox" label="INPUT"></v-checkbox>
                      <div v-if="inputcheckbox">
                        <v-textarea v-model="input" dense filled height="100" no-resize></v-textarea>
                      </div>
                      <select v-model="selectedlanguage">
                        <option value="0">Select a language</option>
                        <option value="50">C (GCC 9.2.0)</option>
                        <option value="54">C++ (GCC 9.2.0)</option>
                        <option value="62">Java (OpenJDK 13.0.1)</option>
                        <option value="51">C# (Mono 6.6.0.161)</option>
                        <option value="71">Python (3.8.1)</option>
                        <option value="63">JavaScript (Node.js 12.14.0)</option>
                        <option value="74">TypeScript (3.7.4)</option>
                        <option value="60">Go (1.13.5)</option>
                      </select>
                      <v-progress-circular
                        indeterminate
                        color="purple"
                        class="mx-2"
                        v-if="runbuttondisable"
                      ></v-progress-circular>
                      <v-btn
                        class="success darken-2 my-1 mx-3"
                        @click="runCode"
                        :disabled="runbuttondisable"
                      >Run</v-btn>
                      <v-btn
                        v-if="person.select == 'CANDIDATE'"
                        width="10"
                        class="success darken-2 my-1"
                        @click="submitCode"
                        :disabled="submitbuttondisable"
                      >Submit</v-btn>
                      <v-btn
                        v-if="person.select != 'CANDIDATE'"
                        width="10"
                        class="success darken-2 my-1"
                        @click="acceptCode"
                        :disabled="acceptbuttondisable"
                      >Accept</v-btn>
                      <v-btn
                        v-if="person.select != 'CANDIDATE'"
                        width="10"
                        class="error darken-2 my-1 mx-3"
                        @click="rejectCode"
                        :disabled="acceptbuttondisable"
                      >Reject</v-btn>
                    </div>
                  </v-expansion-panel-content>
                </v-expansion-panel>

                <!-- Output -->
                <v-expansion-panel width="500" class="ma-2" style="transform: translate(20px);">
                  <v-expansion-panel-header class="blue darken-2 font-weight-bold title">Output</v-expansion-panel-header>

                  <v-expansion-panel-content>
                    <div v-if="!runbuttondisable">
                      <div class="font-weight-black">{{this.submission.outputtext}}</div>
                      <p>{{this.submission.output}}</p>
                      <div class="font-weight-black">{{this.submission.timetext}}</div>
                      <p>{{this.submission.timetext}}{{this.submission.time}}</p>
                      <div class="font-weight-black">{{this.submission.memorytext}}</div>
                      <p>{{this.submission.memorytext}}{{this.submission.memory}}</p>
                    </div>
                  </v-expansion-panel-content>
                </v-expansion-panel>
              </v-expansion-panels>
            </div>
          </v-flex>
          <!-- chatbox -->

          <v-flex lg3 sm4>
            <v-card
              class="elevation-12"
              color="primary lighten-3"
              height="550px"
              style="transform:translate(-32%);"
              min-width="370"
            >
              <v-toolbar dark color="primary darken-1">
                <v-toolbar-title>Chat</v-toolbar-title>
              </v-toolbar>
              <v-card-text>
                <v-list
                  v-chat-scroll
                  ref="chat"
                  id="logs"
                  style="height:365px ;overflow-y: scroll;"
                >
                  <template v-for="(item, index) in chathistory.logs">
                    <div
                      :key="index"
                      class="indigo--text font-weight-black"
                      v-if="chathistory.handle[index]!=''"
                    >
                      <v-icon v-if="chathistory.handle[index] == person.handle">done</v-icon>
                      <v-icon color="blue darken-3" class="mx-2 ma-1 font-weight-black">android</v-icon>
                      {{chathistory.handle[index]}}
                    </div>
                    <v-subheader
                      v-if="item && (chathistory.highlight[index] == true)"
                      :key="index"
                      class="font-italic red--text font-weight-black"
                    >{{ item }}</v-subheader>
                    <v-subheader
                      v-else-if="item"
                      :key="index"
                      class="black--text font-weight-medium"
                    >{{ item }}</v-subheader>
                  </template>
                </v-list>
              </v-card-text>
              <v-card-actions>
                <v-form @submit.prevent="submit">
                  <div style="width:120%">
                    <div v-if="!this.highlight">
                      <v-text-field v-model="msg" label="Message" single-line solo-inverted style></v-text-field>
                    </div>
                    <div v-else>
                      <v-text-field
                        v-model="msg"
                        label="Message"
                        single-line
                        solo-inverted
                        style
                        class="font-italic font-weight-black"
                      ></v-text-field>
                    </div>
                  </div>
                  <v-layout align-center justify-end style="transform:translate(50%,-180%);">
                    <v-btn fab dark small color="primary" type="submit">
                      <v-icon dark>send</v-icon>
                    </v-btn>
                  </v-layout>
                </v-form>
              </v-card-actions>
            </v-card>
          </v-flex>
        </v-layout>
      </v-container>
    </v-content>
    <!-- chatbox -->
  </v-app>
</template>


<script>
import { codemirror } from "vue-codemirror";
import socket from "socket.io-client";

import axios from "axios";
import Navbar from "../components/Navbar.vue";
//codemirror imports
export default {
  components: {
    // eslint-disable-next-line vue/no-unused-components
    codemirror,
    Navbar
  },

  data() {
    return {
      panel_array: [1],
      //questiondata
      host: "localhost",
      question: "",
      tmpquestion: "",
      dialog: false,
      io: null,

      //questiondata

      //chatbox data
      chathistory: {
        logs: [],
        highlight: [],
        handle: []
      },
      msg: null,
      highlight: false,
      //chatbox data

      //person data

      person: null,

      //person data

      //code run and submit data

      inputcheckbox: false,
      input: "",
      runbuttondisable: false,
      submitbuttondisable: false,
      acceptbuttondisable: true,
      submitalert: false,
      acceptalert: false,
      rejectalert: false,
      selectedlanguage: 0,
      submission: {
        output: "",
        time: "",
        memory: "",

        outputtext: "",
        timetext: "",
        memorytext: ""
      },

      //code run and submit data

      //codemirror data

      code: "",
      cmOptions: {
        tabSize: 4,
        styleActiveLine: true,
        lineNumbers: true,
        line: true,
        foldGutter: true,
        styleSelectedText: true,
        mode: "text/x-c++src",
        keyMap: "sublime",
        matchBrackets: true,
        autoCloseBrackets: true,
        showCursorWhenSelecting: true,
        theme: "monokai",
        readOnly: false,
        extraKeys: { Ctrl: "autocomplete" },
        hintOptions: {
          completeSingle: false
        }
      }

      //codemirror data
    };
  },
  methods: {
    questionConfirm() {
      this.dialog = false;
      this.question = this.tmpquestion;
      this.tmpquestion = "";
      var ob = {
        question: this.question,
        room_code: this.person.room_code
      };
      this.io.emit("question", ob);
    },
    questionCancel() {
      this.dialog = false;
      this.tmpquestion = "";
    },
    submit() {
      if (this.msg == "") {
        return;
      }
      console.log(this.msg);

      var tmpmsg = {
        msg: this.msg,
        highlight: this.highlight,
        handle: this.person.handle,
        room_code: this.person.room_code
      };

      this.io.emit("chat", tmpmsg);
      this.msg = "";
      this.highlight = false;
    },
    runCode() {
      if (this.code == "" || this.selectedlanguage == 0) return;
      var ob = {
        source_code: this.code,
        language_id: this.selectedlanguage,
        stdin: this.input
      };
      var token = "";
      (this.runbuttondisable = true), (this.submitbuttondisable = true);
      axios
        .post(
          "https://api.judge0.com/submissions/?base64_encoded=false&wait=false",
          ob
        )
        .then(res => {
          token = res.data.token;

          //get
          setTimeout(() => {
            axios
              .get(
                `https://api.judge0.com/submissions/${token}?base64_encoded=false`
              )
              .then(res => {
                (this.submission.output = res.data.stdout),
                  (this.submission.time = res.data.time),
                  (this.submission.memory = res.data.memory);
                this.panel_array.push(2);
                (this.runbuttondisable = false),
                  (this.submitbuttondisable = false);
                (this.submission.outputtext = "Output: "),
                  (this.submission.timetext = "Time: "),
                  (this.submission.memorytext = "Memory: ");
              })
              .catch(err => {
                (this.runbuttondisable = false),
                  (this.submitbuttondisable = false);
                console.log(err);
              });
          }, 5000);
        })
        .catch(err => {
          (this.runbuttondisable = false), (this.submitbuttondisable = false);
          console.log(err);
        });
    },

    submitCode() {
      this.$swal({
        title: "Are you sure?",
        text: "You won't be able to revert this!",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "Yes, submit it!"
      }).then(result => {
        if (result.value) {
          this.submitbuttondisable = true;
          this.submitalert = true;
          this.acceptbuttondisable = false;
          var ob = {
            submitalert: true,
            room_code: this.person.room_code,
            submitbuttondisable: this.submitbuttondisable,
            acceptbuttondisable: this.acceptbuttondisable,
            selectedlanguage: this.selectedlanguage
          };

          this.io.emit("submitalert", ob);
          this.cmOptions.readOnly = this.submitbuttondisable;
          this.$swal({
            title: "Submitted!",
            text: "Your code has been submitted to the interviewers.",
            icon: "success"
          });
        }
      });
    },

    acceptCode() {
      this.$swal({
        title: "Are you sure?",
        text: "You won't be able to revert this!",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "Yes, Accept it!"
      }).then(result => {
        if (result.value) {
          this.acceptbuttondisable = true;
          this.acceptalert = true;
          this.submitalert = false;
          var ob = {
            submitalert: this.submitalert,
            room_code: this.person.room_code,
            acceptbuttondisable: this.acceptbuttondisable,
            acceptalert: this.acceptalert
          };

          this.io.emit("acceptalert", ob);
          this.$swal({
            title: "Done!",
            text: "You have Accepted the solution of the Candidate",
            icon: "success"
          });
        }
      });
    },

    rejectCode() {
      this.$swal({
        title: "Are you sure?",
        text: "You won't be able to revert this!",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "Yes, Reject it!"
      }).then(result => {
        if (result.value) {
          this.acceptbuttondisable = true;
          this.rejectalert = true;
          this.submitalert = false;
          var ob = {
            submitalert: this.submitalert,
            room_code: this.person.room_code,
            acceptbuttondisable: this.acceptbuttondisable,
            rejectalert: this.rejectalert
          };

          this.io.emit("rejectalert", ob);
          this.$swal({
            title: "Done!",
            text: "You have Rejected the solution of the Candidate",
            icon: "success"
          });
        }
      });
    }
  },

  created() {
    this.person = JSON.parse(localStorage.getItem("person"));

    if (this.person == null) {
      this.$router.push("/");
      return;
    }

    axios
      .get(
        `http://${this.host}:3000/api/persons/valid/${this.person.handle}/${this.person.room_code}/${this.person.select}`
      )
      .then(() => {})
      // eslint-disable-next-line no-unused-vars
      .catch(err => {
        console.log(err);

        if (err.response.status == 404) {
          localStorage.removeItem("person");
          this.$router.push("/");
          return;
        }
      });

    axios
      .get(`http://${this.host}:3000/api/${this.person.room_code}/chatlogs`)
      .then(res => {
        this.chathistory = res.data;
      })
      .catch(err => {
        console.log(err);
      });

    axios
      .get(`http://${this.host}:3000/api/${this.person.room_code}/code`)
      .then(res => {
        this.code = res.data;
      })

      .catch(err => {
        console.log(err);
      });

    axios
      .get(`http://${this.host}:3000/api/${this.person.room_code}/question`)
      .then(res => {
        this.question = res.data;
      })
      .catch(err => {
        console.log(err);
      });

    axios
      .get(
        `http://${this.host}:3000/api/${this.person.room_code}/submissionstatus`
      )
      .then(res => {
        this.submitalert = res.data.submitalert;
        this.submitbuttondisable = res.data.submitbuttondisable;
        this.acceptbuttondisable = res.data.acceptbuttondisable;
        this.acceptalert = res.data.acceptalert;
        this.rejectalert = res.data.rejectalert;
      })
      .catch(err => {
        console.log(err);
      });

    axios
      .get(
        `http://${this.host}:3000/api/${this.person.room_code}/selectedlanguage`
      )
      .then(res => {
        this.selectedlanguage = res.data;
      })
      .catch(err => {
        console.log(err);
      });

    this.io = socket.connect(`http://${this.host}:3000`);

    this.io.emit("room", this.person.room_code);

    this.io.on("question", data => {
      this.question = data;
    });
    this.io.on("code", data => {
      this.code = data;
    });
    this.io.on("chat", data => {
      this.chathistory.logs.push(data.msg);
      this.chathistory.highlight.push(data.highlight);

      this.chathistory.handle.push(data.handle);
    });

    this.io.on("submitalert", data => {
      this.submitalert = data.submitalert;
      this.acceptbuttondisable = false;
      this.selectedlanguage = data.selectedlanguage;
    });

    this.io.on("acceptalert", data => {
      this.acceptalert = data;
      this.submitalert = false;
      this.acceptbuttondisable = true;
    });

    this.io.on("rejectalert", data => {
      this.rejectalert = data;
      this.submitalert = false;
      this.acceptbuttondisable = true;
    });
  },

  mounted() {
    this.$refs.myCm.codemirror.on("keyup", () => {
      var ob = {
        code: this.code,
        room_code: this.person.room_code
      };
      this.io.emit("code", ob);
    });

    this.$refs.myCm.codemirror.on("dblclick", data => {
      var selected_line = data.getCursor().line;
      var selected_code = data.doc.getLine(selected_line).trim();

      if (selected_code.length > 0) {
        selected_code = selected_line + 1 + ")" + selected_code;
        this.msg = selected_code;
        this.highlight = true;
        //this.io.emit('doubleclicked',selected_code);
      }
    });
  }
};
</script>



<style scoped>
.test {
  max-width: 1000px;
  transform: translate(200px);
}
.panelalign {
  max-width: 60%;
  transform: translate(15%);
}

select {
  /* taken from stack-overflow */

  background: linear-gradient(45deg, transparent 50%, blue 50%),
    linear-gradient(135deg, blue 50%, transparent 50%),
    linear-gradient(to right, skyblue, skyblue);
  background-position: calc(100% - 21px) calc(1em + 2px),
    calc(100% - 16px) calc(1em + 2px), 100% 0;
  background-size: 5px 5px, 5px 5px, 2.5em 2.5em;
  background-repeat: no-repeat;

  /* styling and reset */

  border: thin solid blue;
  font: 300 1em/100% "Helvetica Neue", Arial, sans-serif;
  line-height: 1.5em;
  padding: 0.5em 3.5em 0.5em 1em;

  /* reset */

  border-radius: 0;
  margin: 0;
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
  -webkit-appearance: none;
  -moz-appearance: none;
}
.styled-select {
  background: transparent;
  -webkit-appearance: none;
  width: 100px;
  font-size: 11px;
  border: 0;
  height: 17px;
  position: absolute;
  left: 0;
  top: 0;
}
</style>

