<template>
<td @focus="onFocus($event)"  
  @keyup.ctrl.up="onCtrlUp"
  @keyup.ctrl.down="onCtrlDown"
  @keyup.shift.tab="onBackTab"
  :ref="refName+ '_td'">
  <span v-if="flgDebug>=5">hasFocus: {{hasFocus}} {{rowIdx}} {{col.colIdx}}</span>
    <cDateTime v-if="flgEdit && inputType=='timestamp'"
        v-model="row[col.colName]"
        @close="onBlur"
        zone="UTC"
        :ref="refName"
        @change="$emit('change');"
        @input="$emit('change');"
        :tabindex="tabIndex"
        class="data-element-input"
        />
    <input v-if="flgEdit && (inputType == 'text' || inputType == 'date')" 
            :type="inputType"
            v-model="row[col.colName]"
            @change="$emit('change');"
            @blur="onBlur"
            :tabindex="tabIndex"
            :ref="refName"
            class="data-element-input"
            />
 
    <select v-if="flgEdit && inputType == 'select'" 
          v-model="row[col.colName]" 
          @change="$emit('change')"
          :tabIndex="tabIndex"
          @blur="onBlur"
          :ref="refName"
          class="data-element-input"
          >
      <option value=null>-- select --</option>
      <option v-for="(opt,idx) in col.options" :value="opt.value" :key="idx">{{opt.label}}</option>
    </select>
    <span  v-if="!flgEdit">{{displayVal}}</span>
</td>
</template>
<script>
import moment from "moment";
import cDateTime from "./cDateTime.vue";
const STD_INPUTS = "text|date";

export default {
  name: "td-element",
  props: ["row", "col", "rowIdx", "hasFocus"],
  components: { cDateTime },
  data: function() {
    return {
      flgDebug: 0,
      focusLevel: 0,
      flgFocus: false
    };
  },
  watch: {
    // flgFocus(newVal) {
    //    console.log("flgFocus : " + this.col.colName + " : " + newVal);
    // },
    hasFocus(newVal) {
      let colName = this.col ? this.col.colName : "unknown";
      if (newVal) {
        this.flgFocus = true;
        this.focusMe();
      } else {
        this.flgFocus = false;
      }
      if (this.flgDebug >= 2)
        console.log("hasFocus: " + newVal + "; " + colName);
    }
  },
  computed: {
    flgEdit() {
      if (!this.flgFocus) return false;
      let editable = this.col.editable;
      let editor = false;
      if ("editor" in this.col) {
        editor = this.col.editor;
      }
      if (editable == null || editable == undefined) editable = true;
      if (editor === false || (editor !== undefined && editor !== null))
        editor = true;
      let flgEdit = editable && editor;
      return flgEdit;
    },
    tabIndex() {
      return (this.rowIdx + 1) * 100 + this.col.colIdx;
    },
    colType() {
      return this.col.type || "text";
    },

    inputType() {
      var type;
      switch (this.colType) {
        case "text":
          type = "text";
          break;
        case "timestamp":
          type = "timestamp";
          break;
        case "date":
          type = "date";
          break;
        case "select":
          type = "select";
          break;
        default:
          type = "text";
          break;
      }
      return type;
    },
    refName() {
      return this.tabIndex + "_" + this.col.colName;
    },
    displayVal() {
      const displayField = this.col.displayField
        ? this.col.displayField
        : this.col.colName;
      const baseVal = this.row[displayField];
      let dispVal;
      switch (this.col.type) {
        case "idx":
          dispVal = this.rowIdx;
          break;
        case "timestamp":
          if (baseVal == null) {
            dispVal = null;
          } else {
            var d = new moment(baseVal, "YYYY-MM-DDThh:mm");
            dispVal = d.format("MMM D, YYYY, h:mm A");
          }
          break;
        case "date":
          if (baseVal == null) {
            dispVal = null;
          } else {
            var d = new moment(baseVal, "YYYY-MM-DDThh:mm");
            dispVal = d.format("MM/DD/YYYY");
          }
          break;
        case "select":
          if (baseVal == null) dispVal = baseVal;
          else if (this.col.options) {
            const slctdOpt = this.col.options.find(function(el) {
              return el.value == baseVal;
            });
            if (!slctdOpt) dispVal = "value not found";
            else dispVal = slctdOpt.label;
          }
          break;
        default:
          dispVal = baseVal;
      }
      return dispVal;
    }
  },
  methods: {
    focusMe() {
      // Now working withbacktab function
      this.$nextTick(() => {
        let el = this.$refs[this.refName];
        if (!el) el = this.$refs[this.refName + "_td"];
        if (el.$el) el = el.$el; //vue components
        if (el && document.activeElement == el) return;
        el.focus();
        if (this.flgDebug >= 2)
          console.log("focusMe: focusLevel: " + this.refName);
      });
    },
    onBlur() {
      this.flgFocus = false;
    },
    onFocus($evt) {
      if (!this.flgFocus) {
        this.flgFocus = true;
        this.focusMe();
      }
      this.$emit("focusEl");
    },
    onBackTab() {
      if (this.flgDebug >= 2) console.log("DataElement.handleBackTab");
      this.flgFocus = false;
      this.$emit("backtab");
    },
    onCtrlDown() {
      this.flgFocus = false;
      this.$emit("ctrldown");
    },
    onCtrlUp() {
      this.flgFocus = false;
      this.$emit("ctrlup");
    }
  }
};
</script>

<style scoped>
.data-element-input {
  width: 95%;
}
</style>