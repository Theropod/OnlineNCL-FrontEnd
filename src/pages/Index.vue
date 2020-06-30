<template>
  <q-page class>
    <!-- file selection -->
    <q-card class="my-card bg-grey-1 q-pa-md">
      <div class="row q-gutter-sm justify-between items-center">
        <div class="col-12 col-sm-1 text-h5 q-mt-sm q-mb-sm">文件过滤:</div>
        <div
          v-for="(modelFileFilter,index) in modelFileFilters"
          :key="index"
          class="col-12 col-sm-2 q-qa-sm"
        >
          <q-select
            filled
            v-model="modelFileFilter.selected"
            :options="modelFileFilter.filterValues"
            :label="modelFileFilter.filterName"
            :hint="modelFileFilter.hint"
            @input="updateFilters"
          />
        </div>
        <q-btn color="primary" label="搜索" class="col-12 col-sm-1 q-pa-sm" @click="SearchByFilters" />
        <q-btn
          label="重置"
          @click="onModelFileFilterReset"
          color="red"
          flat
          class="col-12 col-sm-1 q-pa-sm"
        />
      </div>
      <div class="q-ma-sm text-body1">搜索结果:</div>
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
    </q-card>
    <q-separator inset />
    <!-- taskForm and Results -->
    <!-- 4:8 in when > breakpoint "small/sm" and stack vertically when < sm on small devices  -->
    <div class="text-h5 q-ma-md">
      脚本任务
      <q-toggle
        true-value="展开"
        false-value="折叠"
        indeterminate-value="待确定展开状态"
        :label="`全部${allTaskExpanded}`"
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
      :label="index+1 + `. ` + taskForm.label"
      class="text-subtitle1"
      header-class="bg-grey-4 "
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
                        hint="当前脚本仅用于绘制S2S模式T变量数据"
                        label="选择脚本"
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
                          label="运行"
                          :disabled="taskForm.disabled"
                          @click="onScriptSubmit(taskForm)"
                          color="primary"
                        />
                        <q-btn
                          label="重置"
                          @click="onScriptReset(taskForm)"
                          color="primary"
                          flat
                          class="q-ml-sm"
                        />
                        <q-btn
                          label="移除"
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
              <q-inner-loading :showing="taskForm.showRunning">
                <q-spinner-gears color="primary" size="6em" />
              </q-inner-loading>
              <!-- Result Pictures -->
              <div class="text-bold q-ma-sm">{{taskForm.scriptOutputFile}}</div>
              <!-- <q-img :src=" '../statics/data/' + taskForm.scriptOutputFile" spinner-color="white" /> -->
              <q-img
                v-if="taskForm.scriptOutputFile == '' || taskForm.scriptOutputFile == '运行失败'"
                src
                spinner-color="white"
              />
              <q-img
                v-else
                :src="`http://166.111.7.72:5000/online-ncl/ncl-wrapper/getImage?filename=${taskForm.scriptOutputFile}`"
                spinner-color="white"
              />
            </div>
          </div>
        </div>
      </q-card>
    </q-expansion-item>
  </q-page>
</template>

<script>
// a simple way to import data
import drawingscripts from "../statics/data/drawingscripts.json";
// function to retrieve file filters based on selection
function updateModelFileFilters(modelFileFilters) {
  // model: position 0, startTime: position 1, variableName: position 2
  for (let i in modelFileFilters) {
    let selectedFilterValue = modelFileFilters[i].selected;
    if (!selectedFilterValue.length) {
      let url =
        "http://166.111.7.72:5000/online-ncl/model-file-manager/findModelFileFilter?columnName=" +
        modelFileFilters[i].columnName +
        "&model=" +
        modelFileFilters[0].selected +
        "&startTime=" +
        modelFileFilters[1].selected +
        "&variableName=" +
        modelFileFilters[2].selected;
      fetch(url)
        .then(response => response.json())
        .then(data => {
          if (data.status == "success") {
            modelFileFilters[i].filterValues = data.result;
          } else {
            console.log(data);
          }
        });
    } else {
      // only preserve one selection item
      modelFileFilters[i].filterValues = [modelFileFilters[i].selected];
    }
  }
}
// function to retrieve object based on its key
function getObjectByFilename(key, parent) {
  for (let item in parent) {
    if (parent[item].filename == key) {
      return parent[item];
    }
  }
  return null;
}
//
function updateAllTaskExpandedState(taskForms) {
  if (taskForms.length) {
    let has_expanded = false;
    let all_expanded = true;
    for (let i in taskForms) {
      all_expanded = all_expanded && taskForms[i].expanded;
      has_expanded = has_expanded || taskForms[i].expanded;
    }
    if (all_expanded) return "展开";
    else if (!has_expanded) return "折叠";
    else return "待确定展开状态";
    return;
  }
  return "展开";
}

export default {
  name: "Index",
  data() {
    return {
      tickedLabels: [],
      taskForms: [],
      allTaskExpanded: "展开",
      drawingScripts: drawingscripts,
      modelFileFilters: [
        {
          filterName: "model",
          hint: "e.g. S2S_T226",
          columnName: "model",
          filterValues: [],
          selected: ""
        },
        {
          filterName: "startTime",
          hint: "e.g. 20200407",
          columnName: "start_time",
          filterValues: [],
          selected: ""
        },
        {
          filterName: "variableName",
          hint: "e.g. T",
          columnName: "variable_name",
          filterValues: [],
          selected: ""
        }
      ],
      modelFiles: []
    };
  },
  methods: {
    updateFilters() {
      updateModelFileFilters(this.modelFileFilters);
    },
    onModelFileFilterReset() {
      for (let i in this.modelFileFilters) {
        this.modelFileFilters[i].selected = "";
      }
      updateModelFileFilters(this.modelFileFilters);
      this.modelFiles = [];
    },
    SearchByFilters() {
      let modelFileFilters = this.modelFileFilters;
      if (
        modelFileFilters[0].selected.length &&
        modelFileFilters[1].selected.length &&
        modelFileFilters[2].selected.length
      ) {
        let url =
          "http://166.111.7.72:5000/online-ncl/model-file-manager/findModelFile?model=" +
          modelFileFilters[0].selected +
          "&startTime=" +
          modelFileFilters[1].selected +
          "&variableName=" +
          modelFileFilters[2].selected;
        fetch(url)
          .then(response => response.json())
          .then(data => {
            if (data.status == "success") {
              this.modelFiles = data.result;
            } else {
              console.log(data);
            }
          });
      } else {
        this.$q.dialog({
          title: "注意",
          message: "请选择所有筛选项"
        });
      }
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
            info: getObjectByFilename(tickedKey, this.modelFiles),
            selectedScript: this.drawingScripts.scriptNames[0],
            scripts: this.drawingScripts.scriptNames,
            // deep copy
            scriptsInfo: JSON.parse(
              JSON.stringify(this.drawingScripts.scriptsInfo)
            ),
            scriptOutputFile: "",
            showRunning: false,
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
      this.allTaskExpanded = updateAllTaskExpandedState(this.taskForms);
    },
    onAllTaskExpandedToggle(state) {
      if (state == "展开") {
        for (let i in this.taskForms) {
          this.taskForms[i].expanded = true;
        }
      } else if (state == "折叠") {
        for (let i in this.taskForms) {
          this.taskForms[i].expanded = false;
        }
      }
    },
    onTaskFormExpandedChange(state) {
      // update All expanded state
      this.allTaskExpanded = updateAllTaskExpandedState(this.taskForms);
    },
    onScriptSubmit(taskForm) {
      // disable run button
      taskForm.disabled = true;
      // temporarily reject tasks whose model file is not supported by NCL Scripts
      if (
        taskForm.info.model == "S2S_T226" &&
        taskForm.info.variableName == "T"
      ) {
        // remove outputfile
        taskForm.scriptOutputFile = "";
        // generate filename, scriptname, parameters, outputfile for api
        let filename = taskForm.info.path;
        let scriptname = taskForm.selectedScript;
        let scriptpath =
          taskForm.scriptsInfo[
            taskForm.scripts.indexOf(taskForm.selectedScript)
          ].path;
        let parameters =
          taskForm.scriptsInfo[
            taskForm.scripts.indexOf(taskForm.selectedScript)
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

        let url = "http://166.111.7.72:5000/online-ncl/ncl-wrapper/run";
        let postdata = {
          filename: filename,
          scriptname: scriptname,
          scriptpath: scriptpath,
          outputfilename: outputfile,
          parameters: parameters
        };
        // show spinner
        taskForm.showRunning = true;

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
            // disable spinner
            taskForm.showRunning = false;
            if (response.status == "Success") {
              taskForm.scriptOutputFile = response.outputFilename;
              this.$q.dialog({
                title: "结果",
                message: "运行成功！"
              });
            } else {
              taskForm.scriptOutputFile = "运行失败";
            }
          });
        // enable run button
        taskForm.disabled = false;
      }
      else {
        taskForm.scriptOutputFile = "数据不支持，脚本仅用于绘制S2S模式T变量数据";
       }
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
  },
  mounted() {
    updateModelFileFilters(this.modelFileFilters);
  }
};
</script>

<style lang="sass" scoped>
</style>
