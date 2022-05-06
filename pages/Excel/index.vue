<template>
  <div class="app-container" :v-loading="loading">
    <h1>Page Drop Excel</h1>
		<div class="box-drop">
        <div class="drop-content" :class="loading ? 'disable-load' : '' ">
          <div class="box__input" @dragover="dragover" @dragleave="dragleave" @drop="drop">
            <svg class="box__icon" xmlns="http://www.w3.org/2000/svg" width="50" height="43" viewBox="0 0 50 43"><path d="M48.4 26.5c-.9 0-1.7.7-1.7 1.7v11.6h-43.3v-11.6c0-.9-.7-1.7-1.7-1.7s-1.7.7-1.7 1.7v13.2c0 .9.7 1.7 1.7 1.7h46.7c.9 0 1.7-.7 1.7-1.7v-13.2c0-1-.7-1.7-1.7-1.7zm-24.5 6.1c.3.3.8.5 1.2.5.4 0 .9-.2 1.2-.5l10-11.6c.7-.7.7-1.7 0-2.4s-1.7-.7-2.4 0l-7.1 8.3v-25.3c0-.9-.7-1.7-1.7-1.7s-1.7.7-1.7 1.7v25.3l-7.1-8.3c-.7-.7-1.7-.7-2.4 0s-.7 1.7 0 2.4l10 11.6z"></path></svg>
            <input type="file" name="fields[upload][]" id="upload" 
              class="w-px h-px opacity-0 overflow-hidden absolute drop-input" @change="onChange" ref="file" accept=".xlsx, .xls, .csv" />
          
            <label for="upload" class="block cursor-pointer drop-label">
              <div>
                Drop files here or <span class="underline">click here</span> to upload Excel files
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
			<ul class="sheet-tab">
				<li  v-for="(name, index) in listSheet" :key="index" @click="changeSheet(index)">{{ name }}</li>
			</ul>
			<table v-if="tableColumn.length">
				<thead>
					<tr>
						<th v-for="(item, index) in tableColumn" :style="{'width': `${item.width}px`}" :key="index">
							{{item.label.indexOf("__EMPTY") ? item.label : ''}}
						</th>
					</tr>
				</thead>
				<tbody>
					<tr  v-for="(item, index) in dataArr" :key="index">
						<td v-for="(e, index) in tableColumn" :style="{'width': `${e.width}px`}" :key="index">
							{{item[e.prop] ? item[e.prop] : ''}}
						</td>
					</tr>
				</tbody>
			</table>
	</div>
</template>
<script>
export default {
	name: 'Excel',
	data(){
    return {
      fileList: [],
			loading: false,
			dataArr: [], // Table content data array
      // countArr: {}, // Analyze the table data and header to get a cross reference array for user-defined consolidation. For the time being, this article only writes the basis, and does not introduce the automatic consolidation of cells~~My other articles have custom merge implementation methods~
      tableColumn: [], // Table header configuration array
      tableLoading: false, // Whether the table is loading
			listSheet: [],
			activeSheet: 0,
			dataExcel: null,
			file: null
    }
  },
	methods: {
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
			console.log(event);
      this.$refs.file.files = event.dataTransfer.files;
      console.log(this.$refs.file.files );
      this.fileList = [...this.$refs.file.files];
			if (this.fileList.length > 1) {
				console.log(this.fileList);
				alert("don't upload mutifiles");
			} else {
				const file = this.$refs.file.files[0];
				this.importExcel(file);
			}
    },
    remove(e) {
      this.fileList.splice(e, 1);
      this.data =[];
			this.dataArr=[];
			this.tableColumn=[];
    },
		onChange(e) {
			e.preventDefault();
      this.$refs.file.files = e.target.files;
			this.fileList = [...this.$refs.file.files];
			console.log(e);
      this.file = e.target.files[0];
      this.importExcel(this.file)
		},
    /**
     * Import table
     */
    importExcel(e) {
			this.file = e
      const files = e 
      const fileRexget = files.name.toLowerCase();
			console.log(fileRexget);
      if (!files.length === 0) {
				if (files.length > 1) {
					alert("don't upload mutifiles");
				}
        return
      } else if (!/\.(xls|xlsx)$/.test(fileRexget)) {
        return alert("The upload format is incorrect. Please upload xls or xlsx format");
      }
      const fileReader = new FileReader();
      fileReader.onload = ev => {
				console.log(ev);
        try {
          this.dataExcel = ev.target.result;
					
          const XLSX = require("xlsx")
					console.log(XLSX);
          const workbook = XLSX.read(this.dataExcel, {
            type: "binary"
          });
					const totalSheet = workbook.Sheets.length;
					this.listSheet = workbook.SheetNames;
          const wsname = workbook.SheetNames[this.activeSheet]; // Take the first sheet，wb.SheetNames[0] :Take the name of the first sheet in the sheets
          const ws = XLSX.utils.sheet_to_json(workbook.Sheets[wsname]); // Generate JSON table content，wb.Sheets[Sheet名]    Get the data of the first sheet
          const excellist = [];  // Clear received data
          // Edit data
          for (var i = 0; i < ws.length; i++) {
            excellist.push(ws[i]);
          }
          console.log("Read results", excellist); // At this point, you get an array containing objects that need to be processed
					const a = workbook.Sheets[workbook.SheetNames[this.activeSheet]];
          const headers = this.getHeader(a);
          console.log('headers', headers);
					this.setTable(headers, excellist);
        } catch (e) {
          return alert("Read failure!");;
        }
      };
      fileReader.readAsBinaryString(files);
      var input = document.getElementById("upload");
      input.value = "";
    },
		getHeader(sheet) {
      const XLSX = require("xlsx");
      const headers = [];
      const range = XLSX.utils.decode_range(sheet["!ref"]); // worksheet['!ref'] Is the valid range of the worksheet
      let C;
      /* Get cell value start in the first row */
      const R = range.s.r; //Line / / column C
      let i = 0;
      for (C = range.s.c; C <= range.e.c; ++C) {
        var cell =
          sheet[
            XLSX.utils.encode_cell({ c: C, r: R })
          ]; /* Get the cell value based on the address  find the cell in the first row */
        var hdr = "UNKNOWN" + C; // replace with your desired default
        // XLSX.utils.format_cell Generate cell text value
        if (cell && cell.t) hdr = XLSX.utils.format_cell(cell);
        if(hdr.indexOf('UNKNOWN') > -1){
          if(!i) {
            hdr = '__EMPTY';
          }else {
            hdr = '__EMPTY_' + i;
          }
          i++;
        }
        headers.push(hdr);
      }
      return headers;
    },
		changeSheet(index) {
			this.activeSheet = index;
			this.importExcel(this.file)
		},
		setTable(headers, excellist) {
      const tableTitleData = []; // Store table header data
      const tableMapTitle = {}; // Set table content for Chinese and English
      headers.forEach((_, i) => {
        tableMapTitle[_] = "prop" + i;
        tableTitleData.push({
          prop: "prop" + i,
          label: _,
          width: 100
        });
      });
      console.log("tableTitleData", tableTitleData);
      // Mapping table content attribute name is English
      const newTableData = [];
      excellist.forEach(_ => {
        const newObj = {};
        Object.keys(_).forEach(key => {
          newObj[tableMapTitle[key]] = _[key];
        });
        newTableData.push(newObj);
      });
      console.log('newTableData',newTableData);
      this.tableColumn = tableTitleData;
      this.dataArr = newTableData;
    },
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
	.sheet-tab {
		display: flex;
		flex-wrap: wrap;
		text-decoration: none;
		margin-bottom: 20px;
		list-style-type: none;
	}
	.sheet-tab li {
		padding: 5px 15px;
		border-right: 1px solid #3c3c3c;
	}
	.sheet-tab li:last-child {
		border: none;
	}
	.sheet-tab li:hover, .sheet-tab li.active {
		color: seagreen;
		text-decoration: underline;
		box-shadow:
       inset 0 -3em 3em rgba(0,0,0,0.1),
             0 0  0 2px rgb(255,255,255),
             0.3em 0.3em 1em rgba(0,0,0,0.3);
	}
</style>
