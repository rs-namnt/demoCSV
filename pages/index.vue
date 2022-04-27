<template>
  <div class="app-container" v-loading="loading">
    <h1>Page Drop</h1>
    <div>
      <div class="box-drop">
        <div class="flex w-full h-screen items-center justify-center text-center drop-content">
          <div class="p-12 bg-gray-100 border border-gray-300" @dragover="dragover" @dragleave="dragleave" @drop="drop">
            <input type="file" name="fields[assetsFieldHandle][]" id="assetsFieldHandle" 
              class="w-px h-px opacity-0 overflow-hidden absolute drop-input" @change="onChange" ref="file" accept=".xlsx, .xls, .csv" />
          
            <label for="assetsFieldHandle" class="block cursor-pointer drop-label">
              <div>
                Drop files here or <span class="underline">click here</span> to upload CSV files
              </div>
            </label>
            <ul class="mt-4" v-if="fileList.length" v-cloak>
              <li class="text-sm p-1" v-for="(file, index) in fileList" :key="index">
                {{file.name}}
                <button class="btn-remove" type="button" @click="remove(fileList.indexOf(file))">remove</button>
              </li>
            </ul>
          </div>
        </div>
      </div>

      <button type="button" class="btn" @click="saveAsExcel" v-if="fileList.length">export excel</button>
      <button type="button" class="btn" @click="convertToCsv" v-if="fileList.length">export csv</button>

      <div v-if="data.length !== 0" style="width : 100%">
        <table ref="tableCsv" id="mytable" >
          <thead>
          <tr>
            <th v-for="(header, index) in header" :key="index">{{header}}</th>
          </tr>
          </thead>
          <tbody>
            <tr v-for="(value, index) in data" :key="index">
              <td v-for="(content, key) in value" :key="key">
                <span v-if="key !== 'isEdit' && !value.isEdit">{{content}}</span>
                <input v-if="key !== 'isEdit' && value.isEdit" type="text" v-model="value[key]"/>
                <span v-if="key === 'isEdit'">
                  <i class="el-icon-edit" @click="editRow(index)"></i>
                  <i class="el-icon-delete" @click="deleteRow(index)"></i>
                </span>       
                <!-- <input v-model="value[key]" v-if="!isShow" >
                <input v-model="picker">
                <span v-if="!isShow" @change="changeData(content)">{Ơ}</span>
                <p>picker : {{picker}}</p> -->
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- <input type="text" class="form-control" id="picker" v-model="picker" /> 
      <p>picker : {{picker}}</p> -->

    </div>
  </div>
</template>

<script>
import { Parser } from "json2csv";
import * as Encoding from 'encoding-japanese';
export default {
  name: 'indexPage',
  data(){
    return {
      result: null, 
      data:[],
      header:[],
      fileList: [],
      isShow: false,
      picker: '',
      loading: false
    }
  },
  methods: {
    async onChange(e) {
      this.loading = true
      this.fileList = [...this.$refs.file.files];
      const file = e.target.files[0];
      await this.readFile(file)
      this.loading = false
    },
    editRow(index){
      // if (!this.data[index].isEdit) {
      //   this.data[index].isEdit = true
      // } else {
      //    this.data[index].isEdit = false
      // }
      console.log(this.data[index],index);
      this.data[index].isEdit = !this.data[index].isEdit
      this.$forceUpdate()
      
    },
    deleteRow(index){
      this.data.splice(index, 1)
      this.$forceUpdate()
    },
    async readFile(e){
      const file = e
      const reader = new FileReader();
      reader.onload = e => {

        this.result = e.target.result
        let uniArray = Encoding.stringToCode(this.result);
        uniArray = uniArray.slice(1)
        const sjisArray = Encoding.convert(uniArray, 'SJIS','AUTO');
        console.log(this.result);
        console.log(uniArray);
        console.log(sjisArray);
        const reg = /,|\t/
        const lines = this.result.split(/\r\n|\r|\n/).filter((item) => {return item !== ""})

        const header = lines[0].split(reg) 
        const output = lines.slice(1).map(line => {

          // let fields = line.replace(/&/g, "&amp;")
          // .replace(/</g, "&lt;")
          // .replace(/>/g, "&gt;")
          // .replace(/"/g, "&quot;")
          // .replace(/'/g, "&#39;");
          // console.log('123');
          // fields = fields.split(reg)
          let fields = line.split(reg)
          return Object.fromEntries(header.map((h, i) => [h, fields[i]])) // 


        })
        
        console.log(output);
        this.data = output;
        this.header = Object.keys(output[0])
        this.header.push('Custom')
        console.log(this.header);
        this.data.forEach(item => item.isEdit = false)
      }
      
      await reader.readAsText(file);
      this.loading = false
      // this.$refs['file'].reset()
    },
    dragover(event) {
      
      event.preventDefault();
      // console.log("dragover: ", event.preventDefault())
      if (!event.currentTarget.classList.contains('bg-green-300')) {
        event.currentTarget.classList.remove('bg-gray-100');
        event.currentTarget.classList.add('bg-green-300');
      }
      // console.log("dragover: ", event.preventDefault())
      
    },
    dragleave(event) {
      event.preventDefault();
      // console.log("dragleave: ", event)
      event.currentTarget.classList.add('bg-gray-100');
      event.currentTarget.classList.remove('bg-green-300');
      // console.log("dragleave: ", event)
      
    },
    drop(event) {
      event.preventDefault();
      this.$refs.file.files = event.dataTransfer.files;
      
      this.fileList = [...this.$refs.file.files];
      const file = this.$refs.file.files[0];
      this.readFile(file);
      
    },
    remove(e) {
      this.fileList.splice(e, 1);
      this.data =[];
    },
    saveAsExcel(){
      this.isShow = true;
      var uri = 'data:application/vnd.ms-excel;base64,',
      template = '<html xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:x="urn:schemas-microsoft-com:office:excel" xmlns="http://www.w3.org/TR/REC-html40"><head><!--[if gte mso 9]><xml><x:ExcelWorkbook><x:ExcelWorksheets><x:ExcelWorksheet><x:Name>{worksheet}</x:Name><x:WorksheetOptions><x:DisplayGridlines/></x:WorksheetOptions></x:ExcelWorksheet></x:ExcelWorksheets></x:ExcelWorkbook></xml><![endif]--></head><body><table>{table}</table></body></html>',
      base64 = function(s) {
        return window.btoa(unescape(encodeURIComponent(s)))
      },
      format = function(s, c) {
        return s.replace(/{(\w+)}/g, function(m, p) {
          // console.log("c[p]: ", c[p])
          return c[p];
        })
      }
      this.$nextTick(() => {
        var toExcel = document.getElementById("mytable").innerHTML;
        var ctx = {
          worksheet: name || '',
          table: toExcel
        };
        // console.log(toExcel);
        var link = document.createElement("a");
        link.download = "export.xls";
        link.href = uri + base64(format(template, ctx))
        link.click();
        this.isShow = false
      })
    },
    getTitleCSV() {
      const header = this.header.map(item => {
        return {
          value: item,
          label: item
        }
      })
      console.log(header);
    },
    convertToCsv() {
      const headers = this.getTitleCSV();
      const records = this.data; 
      const csvParser = new Parser({ fields: headers, withBOM: true });
      const csvFile = csvParser.parse(records);
      let uniArray = Encoding.stringToCode(csvFile);
      uniArray = uniArray.slice(1)
      const sjisArray = Encoding.convert(uniArray, 'SJIS','AUTO');
      const unit8Array = new Uint8Array(sjisArray);

      const blob = new Blob([unit8Array], { type: "text/csv;charset=Shift_JIS" });
      if (navigator.msSaveBlob) {
        // IE 10+
        navigator.msSaveBlob(blob, filename);
      } else {
        const link = document.createElement("a");
        if (link.download !== undefined) {
          // feature detection
          // Browsers that support HTML5 download attribute
          const url = URL.createObjectURL(blob);
          link.setAttribute("href", url);
          link.setAttribute("charset", "Shift_JIS");
          const fileName = `exportCSV.csv`;
          link.setAttribute('download', fileName);
          link.style.visibility = "hidden";
          document.body.appendChild(link);
          link.click();
          document.body.removeChild(link);
        }
      }
    }
  }
}
</script>
<style>
  .box-drop{
    width: 100%;
    text-align: center;
    height: 100%;
    justify-content: center;
    align-items: center;
    display: flex;
  }
  .drop-content{
    border: 1px solid #e2e8f0;
    background-color: #f7fafc;
  }
  .drop-input{
    width: 1px;
    height: 1px;
    position: absolute;
    overflow: hidden;
    opacity: 0;
  }
  .drop-label{
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    width: 700px;
    height: 90px;
}
  .underline {
    text-decoration: underline;
  }
  .frm-data{
    margin-top: 50px;
  }
  .btn {
    padding: 5px 20px;
    background: dodgerblue;
  }
  .btn-remove{
    margin-left: 15px;
  }
  table {
  font-family: Arial, Helvetica, sans-serif;
  border-collapse: collapse;
  width: 100%;
  max-width: 100%;
  overflow: scroll;
  max-height: 800px;
}

table td, table th {
  border: 1px solid #ddd;
  padding: 8px;
}

table tr:nth-child(even){background-color: #f2f2f2;}

table tr:hover {background-color: #ddd;}

table th {
  padding-top: 12px;
  padding-bottom: 12px;
  text-align: left;
  background-color: #04AA6D;
  color: white;
}
</style>
