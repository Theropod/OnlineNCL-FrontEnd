<template>
  <q-page class="row">
    <!-- 4:8 in when > small and stack vertically on small devices  -->
    <div class="col-12 col-md-5 q-pa-sm">
      <q-card class="my-card q-pa-md">
        <div class="text-h5 q-mt-sm q-mb-sm">File List</div>
        <q-input ref="filter" filled v-model="filter" label=" Filename Filter... ">
          <template v-slot:append>
            <q-icon v-if="filter !== ''" name="clear" class="cursor-pointer" @click="resetFilter" />
          </template>
        </q-input>
        <!-- node key must be unique, using file path as key -->
        <!-- leaves will be lazy loaded, in next version -->
        <q-tree
          :nodes="fileTree"
          ref="qfileTree"
          node-key="key"
          :filter="filter"
          :filter-method="myFilterMethod"
          :tick-strategy="tickStrategy"
          :ticked.sync="tickedLabels"
          @update:ticked="onTickedUpdate"
          :expanded.sync="expanded"
        />
      </q-card>
    </div>
    <div class="col-12 col-md-7 q-pa-sm">
      <q-card class="my-card q-pa-md">
        <div class="q-card-section q-ma-md">
          <div class="text-h5 q-ma-md">Function Script</div>
          <!-- `ticked-${tick}` is ES6 template grammar -->
          <!-- <div v-for="tick in tickedLabels" :key="`ticked-${tick}`"> -->
          <div v-for="(scriptForm,index) in scriptForms" :key="index">
            <q-form class="q-gutter-md q-card q-ma-sm q-pa-sm">
              <!-- form header -->
              <div class="q-gutter-sm scriptform-header">
                <!-- will use drawingscripts.json to bind a selective menu here, and determine form fields baed on selection-->
                <div class="text-h6">{{ scriptForm.label }}</div>
              </div>
              <!-- form body -->
              <div class="row q-gutter-md items-start scriptform-body">
                <!-- script form fields -->
                <div class="col q-gutter-sm q-ml-none q-mt-none">
                  <div class="row">
                    <q-select
                      filled
                      v-model="scriptForm.selectedScript"
                      :options="scriptForm.scripts"
                      label="Select Script Function"
                      style="width: 300px"
                      class="col-12 q-ma-sm"
                    />
                    <div
                      v-for="(parameter,index) in scriptForm.scriptsInfo[scriptForm.scripts.indexOf(scriptForm.selectedScript)].parameters"
                      :key="index"
                      class="col-6 q-ma-sm"
                      style="width: 200px"
                    >
                      <q-input
                        filled
                        v-model="parameter.value"
                        :label="parameter.description"
                        :hint="parameter.hint"
                        :type="parameter.type"
                        :readonly="parameter.readonly"
                      />
                    </div>
                    <!-- buttons -->
                    <div class="row col-12 q-ma-sm items-start scriptform-footer">
                      <q-btn label="Run" :disabled="scriptForm.disabled" @click="onDrawingSubmit(scriptForm)" color="primary" />
                      <q-btn
                        label="Reset"
                        @click="onDrawingReset(scriptForm)"
                        color="primary"
                        flat
                        class="q-ml-sm"
                      />
                      <q-btn
                        label="Remove"
                        @click="onRemove(scriptForm.key)"
                        color="red"
                        flat
                        class="q-ml-sm"
                      />
                    </div>
                  </div>
                </div>
                <!-- file information and script information -->
                <div class="col q-gutter-sm q-mr-none q-mt-none">
                  <div class="text-bold">File Information</div>
                  <p style="white-space: pre;">{{ scriptForm.info }}</p>
                  <div class="text-bold">Script Information</div>
                  <p
                    style="white-space: pre;"
                    class="q-mb-none"
                  >{{ scriptForm.scriptsInfo[scriptForm.scripts.indexOf(scriptForm.selectedScript)].description }}</p>
                </div>
              </div>
            </q-form>
          </div>
        </div>
        <q-separator inset />
        <!-- Result Pictures -->
        <div class="q-card-section q-ma-md">
          <div class="text-h5 q-ma-md">Result</div>
          <div class="div q-ma-sm" v-for="(outputfilename,index) in scriptOutputFiles"
                      :key="index">
              <div class="text-bold q-ma-sm">{{outputfilename}}</div>
              <q-img :src="`http://166.111.7.71:8080/online-ncl/getImage?filename=${outputfilename}`" spinner-color="white" />
          </div>
        </div>
      </q-card>
    </div>
  </q-page>
</template>

<script>
// a simple way to import data
import modeloutput from "../statics/data/modeloutput.json";
import drawingscripts from "../statics/data/drawingscripts.json";

// function to retrieve object based on its key
function getObjectByKey(key, parent) {
  const stack = [];
  let node, ii;
  stack.push(parent);

  while (stack.length > 0) {
    node = stack.pop();
    if (node.key == key) {
      return [node]; // return whatever you want here!!!
    } else if (node.children && node.children.length) {
      for (ii = 0; ii < node.children.length; ii += 1) {
        stack.push(node.children[ii]);
      }
    }
  }
  return null;
}

export default {
  name: "Index",
  data() {
    return {
      fileTree: modeloutput,
      filter: "",
      tickedLabels: [],
      scriptForms: [],
      drawingScripts: drawingscripts,
      scriptOutputFiles: [],
      expanded: ["root"],
      tickStrategy: "leaf"
    };
  },
  methods: {
    myFilterMethod(node, filter) {
      const filt = filter.toLowerCase();
      return node.label && node.label.toLowerCase().indexOf(filt) > -1;
    },
    resetFilter() {
      this.filter = "";
      this.$refs.filter.focus();
    },
    onTickedUpdate(ticked) {
      let scriptFormkeys = [];
      // index to remove
      let scriptForms_toremove = [];
      // keys to add
      let scriptForms_toadd = [];
      for (let i in this.scriptForms) {
        scriptFormkeys.push(this.scriptForms[i].key);
      }
      //tick added
      if (ticked.length > scriptFormkeys.length) {
        for (let i in ticked) {
          if (scriptFormkeys.includes(ticked[i])) {
          } else {
            scriptForms_toadd.push(ticked[i]);
          }
        }
        while (scriptForms_toadd.length) {
          let tickedKey = scriptForms_toadd.pop();
          this.scriptForms.push({
            label: getObjectByKey(tickedKey, this.fileTree[0])[0].label,
            key: tickedKey,
            info: getObjectByKey(tickedKey, this.fileTree[0])[0].info,
            selectedScript: this.drawingScripts.scriptNames[0],
            scripts: this.drawingScripts.scriptNames,
            // deep copy
            scriptsInfo: JSON.parse(
              JSON.stringify(this.drawingScripts.scriptsInfo)
            ),
            disabled:false
          });
        }
      }
      //tick removed
      else if (ticked.length < scriptFormkeys.length) {
        for (let j in scriptFormkeys) {
          if (ticked.includes(scriptFormkeys[j])) {
          } else {
            scriptForms_toremove.push(j);
          }
        }
        while (scriptForms_toremove.length) {
          this.scriptForms.splice(scriptForms_toremove.pop(), 1);
        }
      }
    },
    onDrawingSubmit(scriptForm) {
      // disable run button
      scriptForm.disabled=true;
      // generate filename, scriptname, parameters, outputfile for api
      let filename = scriptForm.key;
      let scriptname = scriptForm.selectedScript;
      let scriptpath =
        scriptForm.scriptsInfo[
          scriptForm.scripts.indexOf(scriptForm.selectedScript)
        ].path;
      let parameters =
        scriptForm.scriptsInfo[
          scriptForm.scripts.indexOf(scriptForm.selectedScript)
        ].parameters;
      // check for empty
      for (let i in parameters) {
        let parameter = parameters[i];
        if (parameter.value == null) {
          alert(parameter.description + " cannot be empty");
          return;
        }
      }
      // outputfile: filename_scriptname_parameters.png
      // filename without extension
      let outputfile = scriptForm.label.substring(
        0,
        scriptForm.label.lastIndexOf(".")
      );
      outputfile = outputfile + "_" + scriptForm.selectedScript;
      // add parameter values
      for (let i in parameters) {
        let parameter = parameters[i];
        if (parameter.value) {
          outputfile = outputfile + "_" + parameter.value;
        }
      }
      outputfile = outputfile.replace(/ /g, "_");

      let url = "http://166.111.7.71:8080/online-ncl/run";
      let postdata = {
        filename: filename,
        scriptname: scriptname,
        scriptpath: scriptpath,
        outputfilename: outputfile,
        parameters: parameters
      };

      fetch(url, {
        method: "POST", // or 'PUT'
        body: JSON.stringify(postdata), // data can be `string` or {object}!
        headers: new Headers({
          "Content-Type": "application/json"
        })
      })
        .then(res => res.json())
        .catch(error => console.error("Error:", error))
        .then(response => {
          if (response.status == "Success") {
            this.scriptOutputFiles.push(response.outputFilename);
            alert("NCL Script Successfully Executed");
          } else {
            alert("Script Failed to get output file!");
          }
        });
        // enable run button
        scriptForm.disabled=false;
    },
    onDrawingReset(scriptForm) {
      let a = 0;
      for (let parameter of scriptForm.scriptsInfo[
        scriptForm.scripts.indexOf(scriptForm.selectedScript)
      ].parameters) {
        parameter.value = null;
      }
      let b = 1;
    },
    onRemove(scriptFormkey) {
      let a = 0;
      this.$refs.qfileTree.setTicked(scriptFormkey, false);
      let b = 1;
    }
  }
};
</script>

<style lang="sass" scoped>
.my-card
  width: 100%
</style>
