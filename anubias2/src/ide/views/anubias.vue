<template>
  <div id="app" :style="appStyle">
    <div id="tabs">
      <div :class="`tab ${iTab === tabIndex?'active':''}`" v-for="(tab,iTab) in tabs" :key="iTab" @click="changeTab(iTab)">
        <i class="ri-close-line close" v-if="iTab !== 0"></i>
        {{tabs[iTab]}}
      </div>
    </div>
    <div id="buttons" v-if="tabIndex === 0">
      <div id="device-selector">
        <span>
          Device:
        </span>
        <select v-model="activeDevice" style="width: 60%">
          <option v-for="(device,i) in ide.devices" :value="i" :key="i">
            {{ device.name }}
          </option>
        </select>
      </div>
      <div>
        <icon-button id="orient" v-model="orient" :list="orients" :no-border-y="true"></icon-button>
      </div>
      <div>
        <buttons id="zooms" v-model="zoom" :list="deviceZoom" :no-border-y="true"></buttons>
      </div>
      <div>
        <i class="ri-play-line"></i>
        <i class="ri-bug-line"></i>
        <i class="ri-terminal-line"></i>
        <i class="ri-wifi-line"></i>
      </div>
    </div>

    <div id="main" :style="mainStyle" v-if="tabIndex === 0">
      <div class="grid" >
        <sidebar></sidebar>
        <div id="device-container">
          <device></device>
        </div>
      </div>
    </div>
    <div v-if="tabIndex === 1" class="other-tabs">
      <blue-print></blue-print>
    </div>
    <div id="components" :class="componentsClass" :style="componentsStyle" v-if="tabIndex === 0">
      <h3 @click="expandComponents">
        <span>
          Components
        </span>
        <i class="ri-checkbox-indeterminate-line" @click="toggleComponentsCollapse"></i>
      </h3>
      <components></components>

    </div >
    <div id="properties" :class="propertiesClass" :style="propertiesStyle" v-if="tabIndex === 0">
      <h3 @click="expandProperties">
        <span>
          Properties
        </span>
        <i class="ri-checkbox-indeterminate-line" @click="togglePropertiesCollapse"></i>
      </h3>
      <div class="properties">

      </div>
    </div>
    <anubias-confirm
        :confirm-text="ide.confirm.text"
        :confirm-title="ide.confirm.title"
        :on-confirm="ide.confirm.onConfirm"
        :on-cancel="ide.confirm.onCancel"
        :enabled="ide.confirm.enabled"
    ></anubias-confirm>
    <div id="logs" :class="logClass">
      <h3 @click="expandLogs">
        Logs
        <i class="ri-checkbox-indeterminate-line" @click="toggleLogsCollapse"></i>
      </h3>
      <div id="logs-content">
        logs here
        {{ project.projectFile }}
        {{ project.isSave ? 'save' : 'not save' }}
        "{{tabIndex}}"
      </div>
    </div>
  </div>
</template>

<script>
import {mapState} from 'vuex';
import {mapGetters} from 'vuex';
import buttons from "../components/buttons.vue";
import iconButtons from "../components/icon-buttons.vue";
import device from "../components/device.vue";
import Store from 'electron-store';
import components from "../components/components-panel.vue";
import sidebar from "../components/sidebar.vue";
import anubiasConfirm from "../components/anubias-confirm.vue";
import bluePrint from "./blue-print.vue";
import BluePrint from "./blue-print.vue";
const storage = new Store();

export default {
  name: "anubias",
  components: {BluePrint, buttons, iconButton: iconButtons, device, components, sidebar, anubiasConfirm},
  data: () => {
    return {
      tabIndex: 0,
      tabs: ['Main' , 'Blue Print',"Blue Print 2"],
      deviceZoom: ['AUTO', '120%', '100%', '75%', '50%'],
      orients: ['<i class="ri-smartphone-line"></i>', '<i class="ri-smartphone-line r90deg"></i>'],
    };
  },
  mounted() {
    // if this not need must remove
    // this.setIdeTitle('AnubiasApp');
    // this.$store.dispatch('setIdeTitle', 'AnubiasApp');
    this.initialLoadProject();

  }, computed: {
    ...mapState(['ide', 'project']),
    ...mapGetters(
        'project', ['getPage']
    ),
    ...mapGetters(
        'ide', ['currentPage']
    ),
    zoom: {
      get() {
        return this.$store.state.ide.device.zoom;
      },
      set(value) {
        this.$store.commit('ide/UPDATE_DEVICE_ZOOM', value)
      }
    },
    activeDevice: {
      get() {
        return this.$store.state.ide.device.active;
      },
      set(value) {
        this.$store.commit('ide/UPDATE_DEVICE_ACTIVE', value)
      }
    },
    orient: {
      get() {
        return this.$store.state.ide.device.orient;
      },
      set(value) {
        this.$store.commit('ide/UPDATE_DEVICE_ORIENT', value)
      }
    },
    componentsClass() {
      if (this.ide.components.collapsed) {
        return 'collapsed';
      }
      return '';
    },
    propertiesClass() {
      if (this.ide.properties.collapsed) {
        return 'collapsed';
      }
      return '';
    },
    logClass() {
      if (this.ide.logs.collapsed) {
        return 'collapsed';
      }
      return '';
    },
    appStyle() {
      let style = '';
      // full height others panel when page closed
      if (this.ide.logs.collapsed) {
        style += 'grid-template-rows: 2em 2em 5fr 5fr 30px;';
      }
      return style;
    },
    mainStyle() {
      let style = '';
      // if right panels collapsed
      if (this.ide.components.collapsed && this.ide.properties.collapsed) {
        style += 'grid-column: 1 / 19;';
      }
      return style;
    },
    propertiesStyle() {
      let style = '';
      // if one of them expanded
      if (this.ide.components.collapsed ^ this.ide.properties.collapsed) {
        style += 'grid-row: 3 / 5;';
      }
      // full height properties when component closed
      if (this.ide.components.collapsed && !this.ide.properties.collapsed) {
        style += 'grid-column: 15 / 19;';
      }
      return style;
    },
    componentsStyle() {
      let style = '';
      // if one of them expanded
      if (this.ide.components.collapsed ^ this.ide.properties.collapsed) {
        style += 'grid-row: 3 / 5;';
      }

      // full height component when properties closed
      if (this.ide.properties.collapsed && !this.ide.components.collapsed) {
        style += 'grid-column: 15 / 19;';
      }
      return style;
    }
  },
  methods: {
    changeTab(index) {
      this.tabIndex = index;
    },
    toggleComponentsCollapse() {
      this.$store.dispatch('ide/toggleComponentsCollapse');
    },
    togglePropertiesCollapse() {
      this.$store.dispatch('ide/togglePropertiesCollapse');
    },
    toggleLogsCollapse() {
      this.$store.dispatch('ide/toggleLogsCollapse');
    },
    expandComponents(e) {
      if (this.ide.components.collapsed && e.target.tagName !== 'I') {
        this.toggleComponentsCollapse();
      }
    },
    expandProperties(e) {
      if (this.ide.properties.collapsed && e.target.tagName !== 'I') {
        this.togglePropertiesCollapse();
      }
    },
    // expand page panel
    expandLogs(e) {
      if (this.ide.logs.collapsed && e.target.tagName !== 'I') {
        this.toggleLogsCollapse();
      }
    },
    initialLoadProject() {
      // check if project

      if (this.$store.state.project.projectFile == '') {

        try {
          if (this.$store.state.project.pages.length === []) {
            this.$store.dispatch('project/loadProject', storage.get('lastCreatedProject'));

          } else {
            // set entry point  to active
            this.$store.dispatch('ide/setActivePage', this.$store.state.project.project.entryPoint);
          }
        } catch (e) {
          console.log(e.message);
          this.$store.dispatch('project/loadProject', storage.get('lastLoadedProject'));
        }
      }

      // check backup
      if (!this.$store.state.ide.disableRestoreProject) {
        if (storage.get('backupProject') != null) {
          // console.log(storage.get('backupProject'));

          setTimeout(function () {
            this.$store.dispatch('ide/showConfirm', {
              onConfirm() {

                this.$store.dispatch('project/restoreProject');
              },
              onCancel() {

              },
              text: 'We have unsaved backup, Do you want to restore it?',
              title: 'Project restore confirm',
            });
          }.bind(this), 1000);
        }

      }
    },
  },
}
</script>

<style scoped>
h3 {
  background: var(--def-bg);
  text-align: center;
  font-weight: 100;
  position: relative;
  overflow: hidden;
}

h3 i {
  position: absolute;
  padding: 0 4px;
  left: 0;
  font-weight: 100;
  transition: var(--transition-duration);
}

h3 i:hover {
  box-shadow: inset 0 0 0 20px var(--lighter-bg);
  color: var(--text-hilight);
}

#app {
  user-select: none;
}

#buttons {
  display: grid;
  grid-template-columns: 4fr 2fr 4fr 5fr;
}

#buttons i {
  padding: 5px 15px;
  font-size: 22px;
  border-left: 1px solid var(--lighter-bg);
  transition: var(--transition-duration);
}

#buttons i:last-child {
  border-right: 1px solid var(--lighter-bg);
}

#buttons i:hover {
  box-shadow: inset 0 0 0 20px var(--lighter-bg);
  color: var(--text-hilight);
}

#buttons div:last-child {
  text-align: end;
}

#device-selector span {
  display: inline-block;
  padding: 5px;
  margin-right: 1em;
}

#device-selector select {
  min-width: 50%;
}

#zooms {
  text-align: end;
  max-width: 20em;
}

#orient {
  width: 6em;
}

#main .grid {
  display: grid;
  grid-template-columns: 300px 9fr;
  height: 100%;
  width: 100%;
  overflow-y: auto;
  overflow-x: hidden;
}


#tabs {
}

#tabs .tab {
  position: relative;
  padding: 5px 25px 3px 7px;
  text-align: center;
  border-right: 1px solid var(--lighter-bg);
  display: inline-block;
}

#tabs .tab:first-child {
  padding: 4px 7px 3px;
  width: 7%;
}

#tabs .tab .close {
  position: absolute;
  right: 0;
  top: 0;
  bottom: 0;
  padding: 4px;
  color: var(--lighter-bg);
}

#tabs .tab:hover .close {
  color: darkred;
}

#tabs .tab.active {
  color: var(--text-hilight);
}


@keyframes choosePage {
  0% {
    transform: scale(1);
  }
  35% {
    transform: scale(.85);
  }
  65% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}
</style>