<template>
<div class="row justify-center">
  <div class="create-group-width">

    <div class="q-mb-md text-grey-5 text-h6 text-weight-light ">
      <div v-if="step == 'request_account_name'"  class="row justify-between  ">
        <div>GROUP NAME</div>
        <div>1/{{number_of_steps}}</div>
      </div>
      <div v-if="step == 'request_settings'"  class="row justify-between  ">
        <div>GROUP SETTINGS</div>
        <div>2/{{number_of_steps}}</div>
      </div>
      <div v-else-if="step == 'request_signature'"  class="row justify-between  ">
        <div>CREATING</div>
        <div class="text-uppercase text-primary">{{new_group_account_name}}</div>
      </div>
    </div>

    <q-carousel
      v-model="step"
      keep-alive
      :navigation="false"
      transition-prev="scale"
      transition-next="scale"
      :swipeable="false"
      animated
      control-color="primary"
      :padding="false"
      :arrows="false"
      height="250px"
      class="bg-transparent"
    >
      <q-carousel-slide :name="`request_account_name`" class="no-padding">
        <div>

          <!-- account input -->
          <div class="rounded-borders overflow-hidden">
            <q-input
              ref="accountinput"
              autocomplete="off"
              outlined
              no-error-icon
              counter
              maxlength="12"
              color="accent"
              bg-color="white"
              v-model="new_group_account_name"
              placeholder="Choose your group name"
              :rules="[
                val => !!val || '* Required',
                isValidAccountName,
                val => val.length == 12 || 'Group name must be min 12  chars.',

                isavailableAccountNameWrapper
              ]"
              @input="account_name_validated = false"
            >
              <template v-slot:prepend>
                <q-icon name="people" />
              </template>
              <template v-slot:append>
                <q-icon
                  v-if="account_name_validated"
                  name="check"
                  color="positive"
                />
              </template>
              <template v-slot:hint>
                <span class="text-grey-8 row" v-if="account_name_validated">
                  <!-- <q-icon name="check"/> -->
                  <span>Account name available!</span>
                </span>
                <span v-else>12 char account name</span>
              </template>
            </q-input>
          </div>
          <!-- account input -->
          <!-- next button -->
          <div class="column justify-center items-center q-mt-lg">
            <q-btn
              color="primary"
              label="next"
              style="width:150px"
              :disabled="!account_name_validated"
              @click="next"
            />
            
          <!-- <router-link to="/manage/piecestest55" tag="a" class="text-link text-caption" style="display:block">
            Go to test group
          </router-link> -->

          <!-- <router-link to="/manage/piecestesbbb" tag="a" class="text-link text-caption" style="display:block">
            Go to invalid group
          </router-link> -->

          </div>
          <!-- next button -->
        </div>
      </q-carousel-slide>


      <q-carousel-slide name="request_settings" class="no-padding text-grey-6">
          <div class="row justify-begin items-center ">
            <span class="text-uppercase">{{new_group_account_name}}</span>
            <q-btn icon="edit" round flat size="sm" @click="step='request_account_name'">
              <q-tooltip
                :delay="300"
                anchor="center right"
                self="center left"
                :offset="[10, 0]"
              >
                Edit Group Name.
              </q-tooltip>
            </q-btn>
          </div>
          <!-- voice toggle -->
          <div class="row">
            <div class="text-grey-6">
              <q-toggle
                v-model="voice_only"
                left-label
                :disable="true"
              >
              <div class="row items-center">
                <q-icon name="img:statics/images/voice.png" class="q-mr-xs"/>
                Voice Only
              </div>
              </q-toggle>
              <q-tooltip
                :delay="500"
                anchor="center right"
                self="center left"
                :offset="[0, -5]"
              >
                Voice only groups will be available when Voice has launched. You'll be able to switch an existing group to voice only.
              </q-tooltip>
            </div>
          </div>
          <!-- voice toggle -->
          
          <div class="column justify-center items-center q-mt-md">
            <q-btn
              color="primary"
              label="create"
              style="width:150px"
              :disabled="!account_name_validated"
              @click="createGroup"
            />
            <!-- <q-btn @click="deploycontract" /> -->
            
          <!-- <router-link to="/manage/piecestest55" tag="a" class="text-link text-caption" style="display:block">
            Go to test group
          </router-link> -->
          </div>
      </q-carousel-slide>
      <q-carousel-slide :name="`request_activation`" class="no-padding">
        <div class="column items-center full-height text-grey-6 q-pt-md">
          account created
            <q-btn
              color="primary"
              label="activate"
              style="width:150px"
              :disabled="!account_name_validated"
              @click="activateGroup"
            />
        </div>
      </q-carousel-slide>
      <q-carousel-slide :name="`request_signature`" class="no-padding">
        <div class="column items-center full-height text-grey-6 q-pt-md">
          <q-spinner color="primary" size="40px" @click="step='request_account_name'" class="cursor-pointer" />
          <div class="q-mt-md">Waiting for Signature</div>
        </div>
      </q-carousel-slide>
      <q-carousel-slide :name="`group_created`" class="no-padding">
        <div class="column items-center full-height text-grey-6 q-pt-md">
          <q-icon color="primary" size="40px" name="check" />
          <div class="q-mt-xs">Group created successful</div>
          <q-btn label="visit group" color="secondary" class="q-mt-md" :to="`./manage/${new_group_account_name}`" />
        </div>
      </q-carousel-slide>
    </q-carousel>
  </div>
  <wasm-compiler ref="wasm_compiler"/>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import newGroup from "components/new-group";
import wasmCompiler from "components/wasm-compiler";

import {
  isValidAccountName,
  isAvailableAccountName
} from "../imports/validators";

export default {
  name: "create",
  components: {
    newGroup,
    wasmCompiler
  },
  props:{
    prefill:{
      type: Object,
      default:() =>{
        return {
          group_account_name: ""
        }
      }
    }
  },
  data() {
    return {
      number_of_steps:3,
      step: 'request_account_name', //request_account_name, request_signature
      new_group_account_name: "",
      account_name_validated: false,
      voice_only: false,
      wasmhex: '',
      abihex: ''
    };
  },
  computed: {
    ...mapGetters({
      getAccountName: "ual/getAccountName",
      getAppConfig: "app/getAppConfig"
    })
  },
  methods: {
    isValidAccountName,
    isAvailableAccountName,
    async isavailableAccountNameWrapper(v) {
      const test = await isAvailableAccountName(v);
      console.log(test);
      if (test === true) {
        this.account_name_validated = true;
        return true;
      } else {
        this.account_name_validated = false;
        return test;
      }
    },
    async activateGroup(){
      // alert('creating groups is disabled');
      // return;
      this.deploycontract(this.new_group_account_name);

    },
    async createGroup(){
      // alert('creating groups is disabled');
      // return false;
      this.step="request_signature";
      let create_group = {
        account: this.getAppConfig.groups_contract,
        name: "creategroup",
        data: {
          groupname: this.new_group_account_name,
          creator: this.getAccountName
        }
      };

      let res = await this.$store.dispatch("ual/transact", { actions: [create_group], disable_signing_overlay: true });
      if(res && res.transactionId && res.status == "executed"){
        this.step="request_activation";
      }
      else{
        return false;
      }

    },
    async deploycontract(new_group){
      this.step="request_signature";

      let setcode = {
        account: "eosio",
        name: "setcode",
        data: {
          account: new_group,
          vmtype: 0,
          vmversion: 0,
          code: this.wasmhex
        },
        authorization: [{actor: new_group, permission: 'active'}]
      }

      let setabi = {
        account: "eosio",
        name: "setabi",
        data: {
          account: new_group,
          abi: this.abihex
        },
        authorization: [{actor: new_group, permission: 'active'}]
      }

      let activate = {
        account: this.getAppConfig.groups_contract,
        name: "activate",
        data: {
          groupname: new_group,
          creator: this.getAccountName
         }
      }


      let res = await this.$store.dispatch("ual/transact", { actions: [ setabi, setcode, activate], disable_signing_overlay: true });
      if(res && res.transactionId && res.status == "executed"){
        this.step="group_created";
        return true;
      }
      else{
        return false;
      }


    },
    async next(){
      this.step="request_settings";
      if(this.wasmhex == '' || this.abihex == ''){
        this.wasmhex = (await this.$refs.wasm_compiler.loadRemoteWasm('https://raw.githubusercontent.com/eosgroups/group/master/group.wasm')).wasm;
        
        this.abihex = await this.$refs.wasm_compiler.loadRemoteAbi('https://raw.githubusercontent.com/eosgroups/group/master/group.abi');

      }

    }
  },
  async mounted(){
    if(this.prefill.group_account_name){
      this.new_group_account_name = this.prefill.group_account_name;
    }    
  },
  watch: {
    new_group_account_name: function() {
      console.log(this.$refs.accountinput.hasError);
    }
  }
};
</script>
<style>
.create-group-width {
  width: 300px;
}
.bg-transparent {
  background: transparent;
}
</style>
