<template>
  <div class="app-container" :v-loading="loading">
    <h1>Page Drop CSV</h1>
    <div>
      <div class="box-drop">
        <div class="drop-content" :class="loading ? 'disable-load' : '' ">
          <div class="box__input" @dragover="dragover" @dragleave="dragleave" @drop="drop">
            <svg class="box__icon" xmlns="http://www.w3.org/2000/svg" width="50" height="43" viewBox="0 0 50 43"><path d="M48.4 26.5c-.9 0-1.7.7-1.7 1.7v11.6h-43.3v-11.6c0-.9-.7-1.7-1.7-1.7s-1.7.7-1.7 1.7v13.2c0 .9.7 1.7 1.7 1.7h46.7c.9 0 1.7-.7 1.7-1.7v-13.2c0-1-.7-1.7-1.7-1.7zm-24.5 6.1c.3.3.8.5 1.2.5.4 0 .9-.2 1.2-.5l10-11.6c.7-.7.7-1.7 0-2.4s-1.7-.7-2.4 0l-7.1 8.3v-25.3c0-.9-.7-1.7-1.7-1.7s-1.7.7-1.7 1.7v25.3l-7.1-8.3c-.7-.7-1.7-.7-2.4 0s-.7 1.7 0 2.4l10 11.6z"></path></svg>
            <input type="file" name="fields[assetsFieldHandle][]" id="assetsFieldHandle" 
              class="w-px h-px opacity-0 overflow-hidden absolute drop-input" @change="onChange" ref="file" accept=".xlsx, .xls, .csv" />
          
            <label for="assetsFieldHandle" class="block cursor-pointer drop-label">
              <div>
                Drop files here or <span class="underline">click here</span> to upload CSV files
              </div>
            </label>
            <ul class="mt-4" v-if="fileList.length" v-cloak>
              <li class="item-update" v-for="(file, index) in fileList" :key="index">
                {{file.name}}
                <button class="button-4" type="button" @click="remove(fileList.indexOf(file))">remove</button>
              </li>
            </ul>
          </div>
        </div>
      </div>
      <div class="wrap-button">
        <button type="button" class="button-15" @click="saveAsExcel" v-if="fileList.length">export excel</button>
        <button type="button" class="button-16" @click="convertToCsv" v-if="fileList.length">export csv</button>
      </div>
      <div v-if="data.length !== 0" class="table-scroll">
        <table class="wrap-table" ref="tableCsv" id="mytable" >
          <thead>
          <tr>
            <th v-for="(header, index) in header" :key="index">{{header}}</th>
          </tr>
          </thead>
          <tbody>
            <tr v-for="(value, index) in data" :key="index">
              <td v-for="(content, key) in value" :key="key">
                <span class="max-content" v-if="key !== 'isEdit' && !value.isEdit">{{content}}</span>
                <input v-if="key !== 'isEdit' && value.isEdit" type="text" v-model="value[key]"/>
                <span v-if="key === 'isEdit'">
                  <i class="icon-edit" @click="editRow(index)">Edit</i>
                  <i class="icon-delete" @click="deleteRow(index)">Delete</i>
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
import Encoding from 'encoding-japanese';
export default {
  name: 'CSV',
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
        this.data = output;
        this.header = Object.keys(output[0])
        this.header.push('Custom')
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
    outline: 2px dashed #92b0b3;
    outline-offset: -10px;
    transition: outline-offset .15s ease-in-out, background-color .15s linear;
  }
  .drop-input{
    width: 1px;
    height: 1px;
    position: absolute;
    overflow: hidden;
    opacity: 0;
  }
  .table-scroll {
    max-width: 100%;
    max-height: 800px;
    overflow: auto;
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
.button-4 {
  appearance: none;
  background-color: #FAFBFC;
  border: 1px solid rgba(27, 31, 35, 0.15);
  border-radius: 6px;
  box-shadow: rgba(27, 31, 35, 0.04) 0 1px 0, rgba(255, 255, 255, 0.25) 0 1px 0 inset;
  box-sizing: border-box;
  color: #24292E;
  cursor: pointer;
  display: inline-block;
  font-family: -apple-system, system-ui, "Segoe UI", Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji";
  font-size: 14px;
  font-weight: 500;
  line-height: 20px;
  list-style: none;
  padding: 6px 16px;
  position: relative;
  transition: background-color 0.2s cubic-bezier(0.3, 0, 0.5, 1);
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
  vertical-align: middle;
  white-space: nowrap;
  word-wrap: break-word;
}

.button-4:hover {
  background-color: #F3F4F6;
  text-decoration: none;
  transition-duration: 0.1s;
}

.button-4:disabled {
  background-color: #FAFBFC;
  border-color: rgba(27, 31, 35, 0.15);
  color: #959DA5;
  cursor: default;
}

.button-4:active {
  background-color: #EDEFF2;
  box-shadow: rgba(225, 228, 232, 0.2) 0 1px 0 inset;
  transition: none 0s;
}

.button-4:focus {
  outline: 1px transparent;
}

.button-4:before {
  display: none;
}

.button-4:-webkit-details-marker {
  display: none;
}
.box__icon {
  width: 100%;
  height: 80px;
  fill: #92b0b3;
  display: block;
  margin-top: 40px
}
.item-update {
  list-style: none;
}

.button-15 {
  background-image: linear-gradient(#42A1EC, #0070C9);
  border: 1px solid #0077CC;
  border-radius: 4px;
  box-sizing: border-box;
  color: #FFFFFF;
  cursor: pointer;
  direction: ltr;
  display: block;
  font-family: "SF Pro Text","SF Pro Icons","AOS Icons","Helvetica Neue",Helvetica,Arial,sans-serif;
  font-size: 17px;
  font-weight: 400;
  letter-spacing: -.022em;
  line-height: 1.47059;
  min-width: 30px;
  overflow: visible;
  padding: 4px 15px;
  text-align: center;
  vertical-align: baseline;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
  white-space: nowrap;
}

.button-15:disabled {
  cursor: default;
  opacity: .3;
}

.button-15:hover {
  background-image: linear-gradient(#51A9EE, #147BCD);
  border-color: #1482D0;
  text-decoration: none;
}

.button-15:active {
  background-image: linear-gradient(#3D94D9, #0067B9);
  border-color: #006DBC;
  outline: none;
}

.button-15:focus {
  box-shadow: rgba(131, 192, 253, 0.5) 0 0 0 3px;
  outline: none;
}
.button-16 {
  background-image: linear-gradient(#42ec59, #42ec59);
  border: 1px solid #0077CC;
  border-radius: 4px;
  box-sizing: border-box;
  color: #FFFFFF;
  cursor: pointer;
  direction: ltr;
  display: block;
  font-family: "SF Pro Text","SF Pro Icons","AOS Icons","Helvetica Neue",Helvetica,Arial,sans-serif;
  font-size: 17px;
  font-weight: 400;
  letter-spacing: -.022em;
  line-height: 1.47059;
  min-width: 30px;
  overflow: visible;
  padding: 4px 15px;
  text-align: center;
  vertical-align: baseline;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
  white-space: nowrap;
}
.button-16:hover {
  background-image: linear-gradient(#42ec59, #42ec59);
  border-color: #1482D0;
  text-decoration: none;
}

.button-16:active {
  background-image: linear-gradient(#42ec59, #42ec59);
  border-color: #42ec59;
  outline: none;
}

.button-16:focus {
  box-shadow: rgba(131, 192, 253, 0.5) 0 0 0 3px;
  outline: none;
}
.wrap-button {
  display: flex;
  margin-bottom: 20px;
  margin-top: 20px;
}
.wrap-table {
  max-width:200px;
  overflow: hidden;
}
.max-content {
  display: inline-block;
  width: 180px;
  white-space: nowrap;
  overflow: hidden !important;
  text-overflow: ellipsis;
}
.disable-load {
  pointer-events: none;
}
</style>