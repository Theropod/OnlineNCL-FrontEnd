<template>
  <q-page class>
    <!-- file selection -->
    <q-card class="my-card bg-grey-1">
      <div class="q-pa-md">
        <div class="row q-gutter-sm justify-between items-center">
          <div class="col-12 col-sm-1 text-h5 q-mt-sm q-mb-sm">File Filters:</div>
          <div
            v-for="(modelFileFilter,index) in modelFileFilters"
            :key="index"
            class="col-12 col-sm-3 q-qa-sm"
          >
            <q-select
              filled
              v-model="modelFileFilter.selected"
              :options="modelFileFilter.filterValues"
              :label="modelFileFilter.filterName"
            />
          </div>
          <div class="col-12 col-sm-1 q-qa-sm">
            <q-btn color="primary" label="Search" class="q-ma-sm q-qa-sm" />
          </div>
        </div>
        <div class="q-ma-sm text-body1">Search Results:</div>
        <div class="row q-gutter-sm justify-start">
          <q-list
            dense
            class="rounded-borders col-12 col-sm-3"
            v-for="(modelFile, index) in modelFiles"
            :key="index"
          >
            <q-item tag="label" v-ripple>
              <q-item-section avatar>
                <q-checkbox
                  v-model="tickedLabels"
                  :val="modelFile.filename"
                  @input="onTickedUpdate"
                  :ref="modelFile.filename"
                />
              </q-item-section>
              <q-item-section>
                <q-item-label style="word-wrap:break-word">{{modelFile.filename}}</q-item-label>
                <q-item-label caption>{{modelFile.fileInfo}}</q-item-label>
              </q-item-section>
            </q-item>
          </q-list>
        </div>
      </div>
    </q-card >
    <q-separator inset />
    <!-- taskForm and Results -->
    <!-- 4:8 in when > breakpoint "small/sm" and stack vertically when < sm on small devices  -->
    <div class="text-h5 q-ma-md">
      Task and Result
      <q-toggle
        true-value="Expanded"
        false-value="Folded"
        indeterminate-value="Expanded Not Set"
        :label="`All Task Result ${allTaskExpanded}`"
        v-model="allTaskExpanded"
        class="text-body1"
        @input="onAllTaskExpandedToggle"
      />
    </div>
    <q-expansion-item
      expand-separator
      v-for="(taskForm,index) in taskForms"
      :key="index"
      v-model="taskForm.expanded"
      :label="taskForm.label"
      class="text-subtitle1"
      @input="onTaskFormExpandedChange"
    >
      <q-card flat class="q-ma-sm">
        <div class="my-row row q-gutter-sm">
          <div class="col-12 col-sm-4">
            <div class="q-pa-sm">
              <q-form class="q-gutter-md q-pa-sm">
                <!-- form body -->
                <div class="row q-gutter-md q-mt-none items-start taskForm-body">
                  <!-- script form fields -->
                  <!-- will use drawingscripts.json to bind a selective menu here, and determine form fields baed on selection-->
                  <div class="col q-gutter-sm q-ml-none q-mt-none">
                    <div class="row">
                      <q-select
                        filled
                        v-model="taskForm.selectedScript"
                        :options="taskForm.scripts"
                        hint="select ncl function to run"
                        label="Select Script Function"
                        class="col-7 col-sm-6 q-ma-sm"
                      />
                      <div
                        v-for="(parameter,index) in taskForm.scriptsInfo[taskForm.scripts.indexOf(taskForm.selectedScript)].parameters"
                        :key="index"
                        class="col-6 col-sm-5 q-ma-sm"
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
                      <div class="row col-12 q-ma-sm items-start taskForm-footer">
                        <q-btn
                          label="Run"
                          :disabled="taskForm.disabled"
                          @click="onScriptSubmit(taskForm)"
                          color="primary"
                        />
                        <q-btn
                          label="Reset"
                          @click="onScriptReset(taskForm)"
                          color="primary"
                          flat
                          class="q-ml-sm"
                        />
                        <q-btn
                          label="Remove"
                          @click="onScriptRemove(taskForm.key)"
                          color="red"
                          flat
                          class="q-ml-sm"
                        />
                      </div>
                    </div>
                  </div>
                </div>
              </q-form>
            </div>
          </div>
          <q-separator vertical inset class="q-ma-sm" />
          <div class="col-12 col-sm">
            <div class="q-pa-sm">
              <!-- Result Pictures -->
              <div class="text-bold q-ma-sm">{{taskForm.scriptOutputFile}}</div>
              <q-img :src=" '../statics/data/' + taskForm.scriptOutputFile" spinner-color="white" />
              <!-- <q-img
              :src="`http://166.111.7.71:8080/online-ncl/getImage?filename=${outputfilename}`"
              spinner-color="white"
              />-->
            </div>
          </div>
        </div>
      </q-card>
    </q-expansion-item>
  </q-page>
</template>

<script>
// a simple way to import data
import modeloutput from "../statics/data/modeloutput.json";
import drawingscripts from "../statics/data/drawingscripts.json";

// function to retrieve object based on its key
function getObjectByFilename(key, parent) {
  for (let item in parent) {
    if (parent[item].filename == key) {
      return [item];
    }
  }
  return null;
}

function updateAllTaskExpandedState(taskForms){
  if (taskForms.length) {
    let has_expanded = false;
    let all_expanded = true;
    for (let i in taskForms) {
      all_expanded = all_expanded && taskForms[i].expanded;
      has_expanded = has_expanded || taskForms[i].expanded;
    }
    if (all_expanded) return "Expanded";
    else if (!has_expanded) return "Folded";
    else return "Expanded Not Set";
    return;
  }
  return "Expanded";
}

export default {
  name: "Index",
  data() {
    return {
      tickedLabels: [],
      taskForms: [],
      allTaskExpanded: "Expanded",
      drawingScripts: drawingscripts,
      modelFileFilters: [
        {
          filterName: "model",
          filterValues: ["S2S_T226", "Seasonal_T106"],
          selected: "S2S_T226"
        },
        {
          filterName: "startTime",
          filterValues: ["20200406", "20200407", "20200101"],
          selected: "20200406"
        },
        {
          filterName: "variableName",
          filterValues: ["PRECT", "T"],
          selected: "PRECT"
        }
      ],
      modelFiles: [
        {
          id: 1,
          filename: "daily_bcccsm_2020040600_PRECT.nc",
          model: "S2S_T226",
          startTime: "20200406",
          variableName: "PRECT",
          path: "/Operational_Prediction/S2S_T226/PRECT/20200406",
          fileInfo: "file_info1"
        },
        {
          id: 2,
          filename: "daily_bcccsm_2020040600p01_PRECT.nc",
          model: "S2S_T226",
          startTime: "20200406",
          variableName: "PRECT",
          path: "/Operational_Prediction/S2S_T226/PRECT/20200406",
          fileInfo: "file_info2"
        }
      ]
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
      let taskFormkeys = [];
      // index to remove
      let taskForms_toremove = [];
      // keys to add
      let taskForms_toadd = [];
      for (let i in this.taskForms) {
        taskFormkeys.push(this.taskForms[i].key);
      }
      //tick added
      if (ticked.length > taskFormkeys.length) {
        for (let i in ticked) {
          if (taskFormkeys.includes(ticked[i])) {
          } else {
            taskForms_toadd.push(ticked[i]);
          }
        }
        while (taskForms_toadd.length) {
          let tickedKey = taskForms_toadd.pop();
          this.taskForms.push({
            label: tickedKey,
            // key must not duplicates
            key: tickedKey,
            info: getObjectByFilename(tickedKey, this.modelFiles)[0],
            selectedScript: this.drawingScripts.scriptNames[0],
            scripts: this.drawingScripts.scriptNames,
            // deep copy
            scriptsInfo: JSON.parse(
              JSON.stringify(this.drawingScripts.scriptsInfo)
            ),
            scriptOutputFile:
            "daily_bcccsm_2020040600_T_latlon_plot_21_24_100_CN.png",
            disabled: false,
            expanded: true
          });
        }
      }
      //tick removed
      else if (ticked.length < taskFormkeys.length) {
        for (let j in taskFormkeys) {
          if (ticked.includes(taskFormkeys[j])) {
          } else {
            taskForms_toremove.push(j);
          }
        }
        while (taskForms_toremove.length) {
          this.taskForms.splice(taskForms_toremove.pop(), 1);
        }
      }
      // update All expanded state
      this.allTaskExpanded=updateAllTaskExpandedState(this.taskForms);
    },
    onAllTaskExpandedToggle(state) {
      if (state == "Expanded") {
        for (let i in this.taskForms) {
          this.taskForms[i].expanded = true;
        }
      } else if (state == "Folded") {
        for (let i in this.taskForms) {
          this.taskForms[i].expanded = false;
        }
      }
    },
    onTaskFormExpandedChange(state) {
      // update All expanded state
      this.allTaskExpanded=updateAllTaskExpandedState(this.taskForms);
    },
    onScriptSubmit(taskForm) {
      // disable run button
      taskForm.disabled = true;
      // generate filename, scriptname, parameters, outputfile for api
      let filename = taskForm.key;
      let scriptname = taskForm.selectedScript;
      let scriptpath =
        taskForm.scriptsInfo[taskForm.scripts.indexOf(taskForm.selectedScript)]
          .path;
      let parameters =
        taskForm.scriptsInfo[taskForm.scripts.indexOf(taskForm.selectedScript)]
          .parameters;
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
      let outputfile = taskForm.label.substring(
        0,
        taskForm.label.lastIndexOf(".")
      );
      outputfile = outputfile + "_" + taskForm.selectedScript;
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
      taskForm.disabled = false;
    },
    onScriptReset(taskForm) {
      let a = 0;
      for (let parameter of taskForm.scriptsInfo[
        taskForm.scripts.indexOf(taskForm.selectedScript)
      ].parameters) {
        parameter.value = null;
      }
      let b = 1;
    },
    onScriptRemove(taskFormkey) {
      // let a = 0;
      let checkbox = this.$refs[taskFormkey];
      checkbox[0].toggle();
      // let b = 1;
      // let index = this.tickedLabels.indexOf(taskFormkey);
      // if (index > -1) {
      //   this.tickedLabels.splice(index, 1);
      // }
    }
  }
};
</script>

<style lang="sass" scoped>
</style>
