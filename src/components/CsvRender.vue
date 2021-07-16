<template>
  <v-app>
    <v-toolbar app>        
      <v-flex>
        <v-toolbar-title>Text generator from csv data</v-toolbar-title>
      </v-flex>     
    </v-toolbar>
    <v-content>
      <v-container grid-list-md text-xs-left>
    <v-layout row wrap>
         
    <v-flex xs12>
    <v-card>
      <v-card-title>
        <h3>1. Edit your liquid template</h3>
      </v-card-title>
      <v-card-actions>      
  <v-btn color="secondary"
  small
  v-on:click="nextSample">Next Sample template</v-btn>
  
  <v-card-text>
  <v-icon small>info</v-icon>
      Csv data array is called 'items', to access to header columns use 'headers', to access only to first row you can use 'firstitem'
</v-card-text>
        
        <a target="_blank" rel="noopener noreferrer" href="https://shopify.github.io/liquid/">Liquid template reference</a>
 
      </v-card-actions>
      <v-container>     
        <prism-editor 
 :code="templateLiquid" 
 language="liquid"
 @change="changeTemplate"
 :autostyleLineNumbers="false"
 :line-numbers = "false"
 class="my-editor"
  />          
      </v-container>

    </v-card>
    </v-flex>

<v-flex xs12>
  <v-card>
      <v-card-title>
        <h3>2. Edit your csv data</h3>
      </v-card-title>
       <v-card-actions>
      <v-btn small color="secondary" v-on:click="copyDataFromClipboard" >Paste content from clipboard</v-btn>

    <v-spacer></v-spacer>  
     <a target="_blank" rel="noopener noreferrer" href="https://www.csvjson.com/json2csv">Convert json data to csv data</a>
    

      </v-card-actions>
      <v-container>
                    <codemirror v-model="code" 
    :options="cmOptions"    
    ></codemirror>
      </v-container>
     
    </v-card>
    </v-flex>
 
  
  <v-flex xs12>
  <v-card>
      <v-card-title>
        <h3>3. Get generated text</h3>
      </v-card-title>
       <v-card-actions>
        <v-btn small  color="primary" v-on:click="rendertemplate">Render Liquid template</v-btn>
        
        <v-card-text>
         <v-icon small>info</v-icon>
      The result is automatically copied to clipboard
      </v-card-text>
      </v-card-actions>
      <v-container>
              <v-textarea
              :readonly=true
          v-model="result"
          box
          label="Result"
          height="200px"
          >
          </v-textarea>
      </v-container>     
    </v-card>
    </v-flex>
   
    </v-layout>
   </v-container>
    </v-content>
  </v-app>
</template>

<script>
import { codemirror } from 'vue-codemirror'
import 'codemirror/lib/codemirror.css'
import 'codemirror/theme/base16-dark.css'

import Papa from 'papaparse'

import Liquid from "liquidjs";
import PrismEditor from "vue-prism-editor";
//import initialData from "raw-loader!../assett initialData.txt";
import initialData from "../assets/initialData.tsv";
var engine = new Liquid();
const initialTemplateLiquid =
  '{% for item in items %}\n\
{"key": "{{item.iduser}}", "label": "{{item.iduser}}-{{item.userName | upcase }}", "disabled": false},{% endfor %}';

const sqlTemplateLiquid = `{% for table in items %}
select {{table.iduser}}" from {{table.domain}} where id = "{{table.guidUser}}";{% endfor %}`;

const simpleTemplate = ' {{firstitem.guidUser}} belongs to {{firstitem.userName}}'

const selectInTemplate = `select * from babizagiCatalog where guidObject in ({% for ent in items %}
{% if forloop.first != true %},{% endif %}'{{ent.guidUser}}'{% endfor %}
)`

const copyDllsPsTemplate = `{% for item in items %}
Copy-Item -Path "$bizagiDAOutPut\{{item.dll}}" -Destination "$copyItem\{{item.dll}}" -Force{% endfor %}`

const selectAllFromTable = `{% for item in items %}
select * from [{{item.entsrc}}]{% endfor %}`

const downcaseTemplate = `{% for item in items %}
{{item.guid | downcase }}{% endfor %}`

const createAndInsertDataInTable = `Create table [User] ( {% for header in headers %}
{{header}} varchar(max){% if forloop.last != true %},{% endif %}{% endfor %});

{% for item in items %}INSERT INTO [USER] VALUES({% for header in headers %}{% 
assign columnName = {{header}} %}'{{ item[columnName] }}'{% if forloop.last != true %}, {% endif %}{% endfor %});
{% endfor %}`

export default {
  name: "CsvRender",
  components: {
    PrismEditor,
    codemirror
  },
  methods: {
    rendertemplate: function(event) {
      var resultObject = Papa.parse(this.code, {
        header: true,
      });               
      var data = { 
        firstitem: resultObject.data[0],
        items: resultObject.data,
        headers: Object.keys(resultObject.data[0])
         }      
      engine
        .parseAndRender(this.currentTemplate, data)
        .then(r => {
          this.result = r
          this.$copyText(this.result);
          });
    },
    changeTemplate: function(plain) {
      this.currentTemplate = plain;
    },
    copyDataFromClipboard: function() {
      navigator.clipboard.readText()
      .then(text => {
        this.code = text;
      })
      .catch(err => {        
        console.log('Something went wrong', err);
      });
    },
    nextSample: function(){
      if(this.i>=this.samples.length-1)
      {
        this.i=0;
      }else{
        this.i++;
      }
      this.templateLiquid = this.samples[this.i];
      this.currentTemplate = this.templateLiquid;
    } 
  },
  data() {
    return {  
      i:0,
      samples: [selectInTemplate, createAndInsertDataInTable, selectAllFromTable, downcaseTemplate, copyDllsPsTemplate, simpleTemplate, initialTemplateLiquid, sqlTemplateLiquid],      
      templateLiquid: selectInTemplate,
      currentTemplate: selectInTemplate,
      result: "",            
      code: initialData,     
      cmOptions: {        
        tabSize: 10,
        mode: 'text/javascript',
        theme: 'base16-dark',
        lineNumbers: true,
        line: true
      }
    };
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
textarea {
  width: 100%;
  height: 300px;
}

.my-editor {
  height: 120px;
  font-size: 16pt;
}
</style>
