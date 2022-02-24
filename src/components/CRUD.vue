<style>
.btn-edit{
  margin-right: 15px !important;
  background-color: #0ca0ef !important;
}
.btn-delete{
  background-color: #ff5252 !important;
}
.btn-header{
  background-color:#068d84 !important;
}
.cancel{
  background-color:#030707 !important;
  color: #ffffff !important;
  margin-right: 15px !important;
}
.accept{
  background-color:#44a14e !important;
  color: #ffffff !important;
}
</style>
<template>
    <div>
        <v-card class="mx-auto" >
            <v-container fluid>
                <v-spacer></v-spacer>
                <v-row dense>
                    <v-col cols="6">
                        <v-btn color="danger"
                        dark class="mb-2"
                        @click="dialogForm=true,edit=false,newUser=true"
                        > New User </v-btn>
                    </v-col>
                    <v-col  cols="6">
                        <v-btn
                        dark
                        color="blue-grey"
                        class="btn-header"
                        @click="dialogUpload=true"
                        >
                        Upload File
                        <v-icon
                            right
                            dark
                        >
                            mdi-cloud-upload
                        </v-icon>
                        </v-btn>
                    </v-col>
                </v-row>
                <v-row dense>
                    <v-col cols="12">
                        <v-data-table
                        :headers="headers"
                        :items="listUsers"
                        item-key="name"
                        class="elevation-1"
                        :search="search"
                        >
                            <template v-slot:top>
                                <v-text-field
                                v-model="search"
                                label="Search"
                                single-line
                                hide-details
                                ></v-text-field>
                            </template>
                            <template v-slot:item.id="{ item  }" >
                              <v-btn
                                small
                                color="error"
                                class="btn-edit"
                                @click="editUser(item),edit=true,newUser=false"
                              >
                                Edit
                              </v-btn>
                              <v-btn
                                small
                                color="error"
                                class="btn-delete"
                                @click="deleteDialog(item)"
                              >
                                Delete
                              </v-btn>
                            </template>
                        
                        </v-data-table>
                    </v-col>
                </v-row>
            </v-container>
        </v-card>
        <!-- form upoload -->
        <template>
          <v-row justify="center">
            <v-dialog
              v-model="dialogUpload"
              persistent
              max-width="600px"
            >
              <v-card>
                <v-card-title>
                  <span class="text-h5">Upload File XLSX</span>
                </v-card-title>
                <v-card-text>
                  <v-container>
                    <v-row>
                      <v-col cols="12">
                        <v-file-input
                        v-model="fileUpload"
                        placeholder="upload File xlsx"
                        accept="csv, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
                        prepend-icon=""
                        outlined
                        :clearable="true"
                        class="o-file-search__input"
                        dense>
                        </v-file-input>
                      </v-col>
                    </v-row>
                  </v-container>
                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn
                    color="blue darken-1"
                    text
                    @click="dialogUpload = false"
                  >
                    Cancel
                  </v-btn>
                  <v-btn
                    color="blue darken-1"
                    text
                    :disabled="disabledbtn"
                    @click="uploadFile()"
                  >
                    Upload
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </v-row>
        </template>
        <!-- end form -->
        <!-- dialog delete -->
        <v-dialog
          v-model="dialogDelete"
          persistent
          max-width="290"
        >
          <v-card>
            <v-card-title class="text-h5">
              Warning!
            </v-card-title>
            <v-card-text>
              Deleted user  {{info.userName}} ?
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                class="cancel"
                @click="dialogDelete = false"
              >
                Cancel
              </v-btn>
              <v-btn
                class="accept"
                @click="deleteItem()"
              >
                Yes, Delete
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <!-- dialod delete end -->
        <!-- dialog New user -->
        <v-dialog
          v-model="dialogForm"
          persistent
          max-width="600px"
        >
          <v-card>
            <v-card-title>
              <span class="text-h5">User Data</span>
            </v-card-title>
            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12">
                    <v-text-field
                      label="User ID"
                      required
                      v-model="info.userId"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12">
                    <v-text-field
                      label="User Name"
                      v-model="info.userName"
                      required
                    ></v-text-field>
                  </v-col>
                  <v-col cols="4">
                    <v-text-field
                      label="Date"
                      type="date"
                      v-model="info.date"
                      required
                    ></v-text-field>
                  </v-col>
                  <v-col cols="4">
                    <v-text-field
                      label="HH:mm"
                      v-model="info.punchIn"
                      required
                    ></v-text-field>
                  </v-col>
                  <v-col cols="4">
                    <v-text-field
                      label="HH:mm"
                      v-model="info.punchOut"
                      required
                    ></v-text-field>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              
              <v-btn
                class="cancel"
                @click="dialogForm = false,info={}"
              >
                Cancel
              </v-btn>
              <v-btn
                class="accept"
                @click="guardarInfo()"
              >
                Save
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <!-- dialog end form -->
        
    </div>
</template>
<script>
  import axios from "axios";
  // import { mdiVuetify } from '@mdi/js';
  const API_URL = "http://localhost:3000/api/";
  export default {
    data: () => ({
      // return {
        disabledbtn: false,
        fileUpload:null,
        dialogUpload: false,
        dialogDelete: false,
        dialogForm: false,
        search: '',
        calories: '',
        listUsers: [],
        info:{},
        edit:false,
        newUser:false,
      // }
    }),
    computed: {
      headers () {
        return [
          {
            text: 'User ID',
            align: 'start',
            sortable: false,
            value: 'userId',
          },
          { text: 'User Name', value: 'userName' },
          { text: 'Date', value: 'date' },
          { text: 'Punch In', value: 'punchIn' },
          { text: 'Punch Out', value: 'punchOut' },
          { text: 'Actions', value: 'id' },
        ]
      },
    },
    async created() {
        this.getListaUsers();
    },
    methods: {
      async getListaUsers (){
        await fetch(API_URL+'getUsers')
        .then(response => response.json())
        .then(result => {
          console.log('result',result);
          this.listUsers = result.response;
        });
      },
      async uploadFile(){

        const formData = new FormData();
        formData.append('file', this.fileUpload); ///archivo ajunto
        await axios
        .post(API_URL+"fileUpload", formData)
        .then(res => {
          console.log('res',res);
          if(res.data.response){
            this.listUsers = res.data.response;
          }
          this.dialogUpload = false;
        })
        .catch(err => {
          console.log(err);
        });
      },
      deleteDialog(item){
        console.log(item);
        this.info = item;
        this.dialogDelete = true;
      },
      async deleteItem(){
        await axios
        .post(API_URL+'deleteUser', {id:this.info.id})
        .then(res => {
          this.dialogDelete = false;
          // eliminamos registro
          let userList = this.listUsers.filter(user => user.id != this.info.id);
          this.listUsers = userList;
          this.info={};
        })
        .catch(err => {
          console.log(err);
        });
      },
      async guardarInfo(){
        console.log(['item',this.info,this.edit,this.newUser]);
        if(this.newUser){ // nuevo usuario
          await axios
          .post(API_URL+'createUser', this.info)
          .then(res => {
            console.log(res);
            this.dialogForm = false;
            this.listUsers = res.data.response;
            this.info={};
          })
          .catch(err => {
            console.log(err);
          });
        }else{ // editar usuario
          await axios
          .post(API_URL+'EditUser', this.info)
          .then(res => {
            console.log(res);
            this.dialogForm = false;
            this.listUsers = res.data.response;
            this.info={};
          })
          .catch(err => {
            console.log(err);
          });
        }
      },
      async editUser(item){
        this.info = item;
        this.dialogForm = true;
      }
    },
  }
</script>