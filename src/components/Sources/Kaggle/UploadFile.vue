<template>
  <div>
    <div class="margin-bottom">
      <input type="file" id="file" name="file" @change="loadFile($event)">
    </div>
    <div v-if="isFileLoaded()">
      <div class="margin-bottom">
        <input type="checkbox" id="saveToDatabase" name="saveToDatabase" v-model="saveToDatabase">
        <label for="saveToDatabase">Save to database</label>
      </div>
      <br>
      <div class="margin-bottom--submit">
        <button @click="uploadFile()">Submit</button>
      </div>
    </div>
  </div>
  <div class="margin-bottom" v-if="errors.length">
    <ul>
      <li v-for="(error, index) in errors" :key="index">
        {{ error }}
      </li>
    </ul>
  </div>
  <div class="margin-bottom" v-if="fileUploaded">
    <div class="margin-bottom bold">
      Avg price: {{ avgPrice }}
    </div>
    <div class="margin-bottom bold">
      Total houses sold: {{ totalHousesSold }}
    </div>
    <div class="margin-bottom--submit bold">
      No of crimes in 2011: {{ numberOfCrimesIn2011 }}
    </div>
    <div class="bold">
      Avg price per year in London area
    </div>
    <div v-for="(avgPrice, year) in avgPricePerYearInLondonArea" :key="year">
      {{ year }}: {{ avgPrice }}
    </div>
  </div>
</template>

<script>
import _ from 'lodash';
import axios from 'axios';
import { KAGGLE_SOURCE } from "@/constants";

export default {
  name: 'UploadFile',
  data() {
    return {
      file: {},
      avgPrice: 0,
      totalHousesSold: 0,
      numberOfCrimesIn2011: 0,
      avgPricePerYearInLondonArea: [],
      saveToDatabase: false,
      fileUploaded: false,
      validFileExtensions: ['csv'],
      errors: []
    }
  },
  methods: {
    loadFile(e) {
      let vm = this;
      vm.resetData();
      vm.validateFileLoad(e);

      if (!vm.errors.length) {
        vm.file = e.target.files[0]
      }
    },
    uploadFile() {
      let vm = this;
      vm.validateFileUpload();

      if (vm.errors.length) {
        return;
      }

      let saveToDatabase = 0;

      if (vm.saveToDatabase) {
        saveToDatabase = 1;
      }

      let data = new FormData();
      data.append('file', vm.file);
      data.append('source', KAGGLE_SOURCE);
      data.append('saveToDatabase', saveToDatabase);
      axios.post("/api/upload-file", data).then((response) => {
        vm.resetData();
        vm.setStatistic(response.data.data);
      })
          .catch(function (error) {
            vm.handleServerErrors(error.response.data.errors);
          })
    },
    handleServerErrors(error) {
      let vm = this;
      let errorKey = Object.keys(error)[0];

      _.forEach(error[errorKey], (error) => {
        vm.errors.push(error);
      });
    },
    setStatistic(data) {
      let vm = this;
      vm.avgPrice = data.avgPrice;
      vm.totalHousesSold = data.totalHousesSold;
      vm.numberOfCrimesIn2011 = data.numberOfCrimesIn2011;
      vm.avgPricePerYearInLondonArea = JSON.parse(data.avgPricePerYearInLondonArea);
      vm.fileUploaded = true;
    },
    resetData() {
      let vm = this;
      vm.avgPrice = 0;
      vm.totalHousesSold = 0;
      vm.numberOfCrimesIn2011 = 0;
      vm.avgPricePerYearInLondonArea = [];
      vm.fileUploaded = false;
      vm.saveToDatabase = false;
      vm.file = {};
      vm.errors = [];
    },
    validateFileLoad(e) {
      let vm = this;
      let fileExtension = e.target.value.split('.').pop();

      if (!vm.validFileExtensions.includes(fileExtension)) {
        e.target.value = null;
        vm.errors.push('Allowed file formats include ' + vm.validFileExtensions.toString());
      }
    },
    validateFileUpload() {
      let vm = this;

      if (!vm.isFileLoaded()) {
        vm.errors.push('Please load file first.');
      }
    },
    isFileLoaded() {
      let vm = this;
      return vm.file instanceof File;
    }
  }
}
</script>

<style>
.margin-bottom {
  margin-bottom: 10px;
}

.margin-bottom--submit {
  margin-bottom: 40px;
}

.bold {
  font-weight: bold
}
</style>
