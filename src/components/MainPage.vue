<template>
  <div class="hello">
    <b-button class="clearButton" block variant="outline-danger" v-show="showClearButton" @click="clearScreen()">Clear</b-button>
    <div class="inputFiles">
      <b-form-group label="Select File 1:" label-cols-sm="2" label-size="sm">
        <b-form-file 
          v-model="file1"
          :state="Boolean(file1)"
          placeholder="Choose a file or drop it here..."
          id="file1" 
          size="sm"
          accept=".csv"
          > 
        </b-form-file>
        <div class="validationError" v-if="!$v.file1.required"></div>
      </b-form-group>
      <b-form-group label="Select File 2:" label-cols-sm="2" label-size="sm">
        <b-form-file 
          v-model="file2"
          :state="Boolean(file2)"
          placeholder="Choose a file or drop it here..." 
          id="file2"
          size="sm"
          accept=".csv"
          >
        </b-form-file>
        <div class="validationError" v-if="!$v.file1.required"></div>
      </b-form-group>
        <b-button block variant="outline-success" @click="sendFiles" :disabled="checkCompareValidations()">Compare</b-button>
    </div>
    <div class="compareFiles" v-show="!showCompareResults">
      <b-card-group deck>
        <b-card
          :header=file1.name
        >
          <b-card-text>Total Records: {{totalRecordsFile1}}</b-card-text>
          <b-card-text>Matching Records: {{matchingRecordsFile1}}  </b-card-text>
          <b-card-text>Unmatched Records: {{totalUnmatchedRecordsFile1}}</b-card-text>
        </b-card>

        <b-card :header=file2.name>
          <b-card-text>Total Records: {{totalRecordsFile2}} </b-card-text>
          <b-card-text>Matching Records: {{matchingRecordsFile2}} </b-card-text>
          <b-card-text>Unmatched Records: {{totalUnmatchedRecordsFile2}} </b-card-text>
        </b-card>
      </b-card-group>
      <b-button block variant="outline-info" class="unmatchedReportButton" :disabled="this.totalUnmatchedRecordsFile2 == 0 && this.totalUnmatchedRecordsFile1 == 0" @click="showUnmatchedReportTables()">Unmatched Report</b-button>
    </div>
    <div class="unmatchedReportDiv" v-show="!showUnmatchedReport">
      <b-table
        caption-top
        sticky-header
        no-border-collapse
        small
        head-variant="light"
        responsive
        :items="unmatchedRecordsFile1"
        :fields="fieldsUnmatchedReport"
        ref="unmatchedReportTableFile1"
        id="unmatchedReportTableFile1"
        class="unmatchedReportTableFile1Class"
      >
        <template #table-caption>{{ file1.name }}</template>
      </b-table>
      <b-table
        caption-top
        sticky-header
        no-border-collapse
        small
        head-variant="light"
        responsive
        :items="unmatchedRecordsFile2"
        :fields="fieldsUnmatchedReport"
        ref="unmatchedReportTableFile2"
        id="unmatchedReportTableFile2"
        class="unmatchedReportTableFile2Class"
      >
        <template #table-caption>{{ file2.name }}</template>
      </b-table>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import { required } from 'vuelidate/lib/validators'
export default {
  name: 'MainPage',
  props: {
    msg: String
  },
  data : function() {
    return {
        file1: '',
        file2: '',
        showClearButton: false,
        showUnmatchedReport: true,
        showCompareResults: true,
        totalRecordsFile1: '',
        matchingRecordsFile1: '',
        totalUnmatchedRecordsFile1: '',
        totalRecordsFile2: '',
        matchingRecordsFile2: '',
        totalUnmatchedRecordsFile2: '',
        unmatchedRecordsFile1: [],
        unamtchedRecordsFile2: [],
        fieldsUnmatchedReport: [
                // {key: 'profileName', label: 'Profile Name' },
                {key: 'connection', label: 'Connection', sortable: true},
                {key: 'transactionDate', label: 'Tran. Date', sortable: true},
                {key: 'walletReference', label: 'Wallet' },
                {key: 'transactionAmount', label: 'Tran. Amount', sortable: true},
                // {key: 'transactionNarrative', label: 'Tran. Narrative' },
                // {key: 'transactionDescription', label: 'Tran. Description' },
                // {key: 'transactionID', label: 'Tran. ID' },
                // {key: 'transactionType', label: 'Tran. Type' },
            ],
    } 
  },
  validations: {
    file1: {
        required
    },
    file2: {
        required
    },
  },
  methods: {
    showUnmatchedReportTables(){
      this.showUnmatchedReport = false;
    },
    makeToast(title, variant, content, delay) {
      this.$bvToast.toast(content, {
          title: title,
          toaster: 'b-toaster-top-right',
          variant: variant,
          solid: true,
          "auto-hide-delay": delay
      })
    },
    clearScreen(){
      this.showCompareResults = true;
      this.showUnmatchedReport = true;
      this.file2 = '';
      this.file1 = '';
      this.totalRecordsFile1 = '';
      this.totalUnmatchedRecordsFile1 = '';
      this.matchingRecordsFile1 = '';
      this.unmatchedRecordsFile1 = [];
      this.totalRecordsFile2 = '';
      this.totalUnmatchedRecordsFile2 = '';
      this.matchingRecordsFile2 = '';
      this.unmatchedRecordsFile2 = [];
      this.showClearButton = false;
    },
    checkCompareValidations (){
      if(this.$v.file1.$invalid || this.$v.file2.$invalid)
        return true;
      else
        return false;
    },

    clearData(){
      this.showCompareResults = true;
      this.showUnmatchedReport = true;
      this.showClearButton = false;
    },

    sendFiles() {
      this.clearData();
      const headers = {
        'Content-Type': '"multipart/form-data"'       
      }
      let formData1 = new FormData();
      formData1.append('csvFile', this.file1);
      formData1.append('csvFile2', this.file2);
      let URL = 'api/uploadFiles';
      let promise = axios.post(URL, formData1, {
        headers: headers
        })
      return promise.then((response) => {
          if(response.data.responseMessage == 'SUCCESS'){
            this.totalRecordsFile1 = response.data.totalNumberFile1;
            this.totalUnmatchedRecordsFile1 = response.data.unmatchedRecordsFile1;
            this.matchingRecordsFile1 = this.totalRecordsFile1 - this.totalUnmatchedRecordsFile1;
            this.unmatchedRecordsFile1 = response.data.clientProfileFile1;
            this.totalRecordsFile2 = response.data.totalNumberFile2;
            this.totalUnmatchedRecordsFile2 = response.data.unmatchedRecordsFile1;
            this.matchingRecordsFile2 = this.totalRecordsFile2 - this.totalUnmatchedRecordsFile2;
            this.unmatchedRecordsFile2 = response.data.clientProfileFile2;
            this.showCompareResults = false;
            this.makeToast('File Comparison Finished', 'primary', response.data.responseMessage, 10000);
            this.showClearButton = true;
          }else{
            this.makeToast('File Comparison Failed', 'warning', response.data.responseMessage, 10000);
          }

        },
        ).catch(err => {
          this.makeToast('File Comparison Failed', 'danger', err, 10000);

        })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .inputFiles {
    width: 50%;
    margin : 0 auto;
    margin-top: 1%;
  }
  .compareFiles {
    width: 50%;
    margin : 0 auto;
    margin-top:1%;
  }
  .unmatchedReportButton {
    margin-top: 1%;
  }
  .unmatchedReportDiv {
    margin-top: 1%;
  }
  .unmatchedReportTableFile1Class{
    float: left;
    width: 48%;
    margin-left:2%;
  }
  .unmatchedReportTableFile2Class{
    float: left;
    width: 48%;
    margin-left:2%;
  }
  .clearButton{
    margin: 0 auto;
    width: 50%;
    margin-top: 1%;
  }
</style>
