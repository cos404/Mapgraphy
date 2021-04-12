<template>
  <div id="app">
    <div id="workplace">
      <div class="title">{{mapTitle}}</div>
      <vue-drag-resize
        id="draggable"
        @resizing="resize"
        @dragging="drag"
        :parent="true"
        :active="draggableActive"
        :draggable="draggable"
        :x="20"
        :y="20"
        :z="1"
        :w="120"
        :min-width="120"
        :h="draggableHeight"
        :handles="['mr', 'ml']"
      >
        <div id="colors">
          <div
            @mouseover="mouseOverPalette(false)"
            @mouseout="mouseOverPalette(true)"
            class="colors-picker"
            :style="{ top: mouse.y, left: mouse.x }"
            v-if="displayPicker"
          >
            <div class="input-group color-picker" ref="colorpicker">
              <span class="input-group-addon color-picker-container">
                <compact-picker v-model="lastPaletteColor" />
              </span>
            </div>
          </div>
          <div class="colors-palette">
              <ul>
                <li
                  v-for="(color, index) in colors"
                  class="color"
                  @click="selectColor(index)"
                  :key="index"
                >
                  <div class="content" :class="{'selected': currentIndexColor == index}">
                    <span class="selectedColor" :style="{backgroundColor: color.hex }"></span>
                    <input
                      @mouseover="mouseOverPalette(false)"
                      @mouseout="mouseOverPalette(true)"
                      type="text"
                      v-model="color.title"
                    />
                  </div>
                </li>
              </ul>
          </div>
        </div>
      </vue-drag-resize>
      <div id="map" @click="coloring">
        <worldMap :width="'1000'" :height="'600'" :viewBox="'0 0 1000 720'" />
      </div>
      <div id="made">
        Created by WorkName<br>
        <input type="text" name="" value="for" />
      </div>
    </div>
    <vue-drag-resize
      id="draggableControll"
      @resizing="resize"
      @dragging="drag"
      :draggable="draggableControll"
      :resizable="false"
      :parent="true"
      :active="false"
      :y="620"
      :x="500"
      :z="1"
      :w="658"
      :min-width="658"
      :h="50"
    >
      <div id="controll">
        <button
          class="btn-reset btn-download"
          @click="saveImage(1280)"
          @mouseover="mouseOverControll(false)"
          @mouseout="mouseOverControll(true)"
        >Download 1280</button>
        <button
          class="btn-reset btn-download"
          @click="saveImage(1920)"
          @mouseover="mouseOverControll(false)"
          @mouseout="mouseOverControll(true)"
        >Download 1920</button>
        <button
          class="btn-reset btn-download"
          @click="saveImage(3840)"
          @mouseover="mouseOverControll(false)"
          @mouseout="mouseOverControll(true)"
        >Download 3840</button>
        <button
          class="btn-reset btn-add-colors"
          @click='add_color'
          @mouseover="mouseOverControll(false)"
          @mouseout="mouseOverControll(true)"
        >+</button>
        <button
          class="btn-reset btn-add-colors"
          @click="remove_color"
          @mouseover="mouseOverControll(false)"
          @mouseout="mouseOverControll(true)"
        >-</button>
        <input
          type="text"
          v-model="mapTitle"
          @mouseover="mouseOverControll(false)"
          @mouseout="mouseOverControll(true)"
        />
        <select v-model="mapSelected">
          <option v-for="(map, index) in MAPS" :key="index" >{{map}}</option>
        </select>
      </div>
    </vue-drag-resize>
  </div>
</template>


<script>

import { Compact } from 'vue-color';
import html2canvas from 'html2canvas';
import VueDraggableResizable from 'vue-draggable-resizable';
import Canvas2Image from './canvas2image';

import Map from './components/world';

const countriesIds = ['AD', 'AE', 'AF', 'AG', 'AI', 'AL', 'AM', 'AO', 'AR', 'AS', 'AT', 'AU', 'AW', 'AX', 'AZ', 'BA', 'BB', 'BD', 'BE', 'BF', 'BG', 'BH', 'BI', 'BJ', 'BL', 'BN', 'BO', 'BM', 'BQ', 'BR', 'BS', 'BT', 'BV', 'BW', 'BY', 'BZ', 'CA', 'CC', 'CD', 'CF', 'CG', 'CH', 'CI', 'CK', 'CL', 'CM', 'CN', 'CO', 'CR', 'CU', 'CV', 'CW', 'CX', 'CY', 'CZ', 'DE', 'DJ', 'DK', 'DM', 'DO', 'DZ', 'EC', 'EG', 'EE', 'EH', 'ER', 'ES', 'ET', 'FI', 'FJ', 'FK', 'FM', 'FO', 'FR', 'GA', 'GB', 'GE', 'GD', 'GF', 'GG', 'GH', 'GI', 'GL', 'GM', 'GN', 'GO', 'GP', 'GQ', 'GR', 'GS', 'GT', 'GU', 'GW', 'GY', 'HK', 'HM', 'HN', 'HR', 'HT', 'HU', 'ID', 'IE', 'IL', 'IM', 'IN', 'IO', 'IQ', 'IR', 'IS', 'IT', 'JE', 'JM', 'JO', 'JP', 'JU', 'KE', 'KG', 'KH', 'KI', 'KM', 'KN', 'KP', 'KR', 'XK', 'KW', 'KY', 'KZ', 'LA', 'LB', 'LC', 'LI', 'LK', 'LR', 'LS', 'LT', 'LU', 'LV', 'LY', 'MA', 'MC', 'MD', 'MG', 'ME', 'MF', 'MH', 'MK', 'ML', 'MO', 'MM', 'MN', 'MP', 'MQ', 'MR', 'MS', 'MT', 'MU', 'MV', 'MW', 'MX', 'MY', 'MZ', 'NA', 'NC', 'NE', 'NF', 'NG', 'NI', 'NL', 'NO', 'NP', 'NR', 'NU', 'NZ', 'OM', 'PA', 'PE', 'PF', 'PG', 'PH', 'PK', 'PL', 'PM', 'PN', 'PR', 'PS', 'PT', 'PW', 'PY', 'QA', 'RE', 'RO', 'RS', 'RU', 'RW', 'SA', 'SB', 'SC', 'SD', 'SE', 'SG', 'SH', 'SI', 'SJ', 'SK', 'SL', 'SM', 'SN', 'SO', 'SR', 'SS', 'ST', 'SV', 'SX', 'SY', 'SZ', 'TC', 'TD', 'TF', 'TG', 'TH', 'TJ', 'TK', 'TL', 'TM', 'TN', 'TO', 'TR', 'TT', 'TV', 'TW', 'TZ', 'UA', 'UG', 'UM_DQ', 'UM_FQ', 'UM_HQ', 'UM_JQ', 'UM_MQ', 'UM_WQ', 'US', 'UY', 'UZ', 'VA', 'VC', 'VE', 'VG', 'VI', 'VN', 'VU', 'WF', 'WS', 'YE', 'YT', 'ZA', 'ZM', 'ZW'];

export default {
  components: {
    compactPicker: Compact,
    worldMap: Map,
    vueDragResize: VueDraggableResizable,
  },
  data() {
    return {
      MAPS: ['world', 'belarus', 'denmark', 'estonia', 'finland', 'france', 'georgia', 'germany', 'greece', 'italy', 'netherlands', 'russia', 'spain', 'ukraine', 'united-kingdom', 'usa'],
      mapSelected: 'world',
      draggableWidth: 0,
      draggableHeight: 49,
      draggableMinHeight: 49,
      draggableTop: 0,
      draggableLeft: 0,
      draggableActive: false,
      draggable: true,
      draggableControll: true,
      mapTitle: 'Map title',
      lastPaletteColor: { hex: '#0C797D' },
      currentIndexColor: 0,
      displayPicker: false,
      countries: { AD: 0 },
      colors: [
        {
          hex: '#000',
          title: 'Color title',
        },
      ],
      mouse: {
        x: 0,
        y: 0,
      },
    };
  },
  watch: {
    lastPaletteColor() {
      this.colors[this.currentIndexColor].hex = this.lastPaletteColor.hex;
    },
    currentIndexColor() {
      this.lastPaletteColor.hex = this.colors[this.currentIndexColor].hex;
    },
    colors: {
      handler() {
        this.colorsUpdated();
      },
      deep: true,
    },
  },
  methods: {
    selectColor(index) {
      this.currentIndexColor = index;
      this.lastPaletteColor = {};
      this.lastPaletteColor.hex = this.colors[index].hex;
      this.showPicker();
    },
    showPicker() {
      document.addEventListener('click', this.documentClick);
      this.displayPicker = true;
    },
    hidePicker() {
      document.removeEventListener('click', this.documentClick);
      this.displayPicker = false;
    },
    documentClick(e) {
      const el = this.$refs.colorpicker;
      const target = e.target;
      const draggable = document.getElementById('draggable');
      const workplace = document.getElementById('workplace');
      const left = e.clientX - draggable.offsetLeft - workplace.offsetLeft - 5;
      const top = e.clientY - draggable.offsetTop - workplace.offsetTop - 5;

      if (!el.contains(target)) this.mouse = { x: `${left}px`, y: `${top}px` };
      if (this.displayPicker && el !== target && !el.contains(target) && target.classList[0] !== 'selectedColor') {
        this.hidePicker();
      }
    },
    add_color() {
      this.colors.push({ hex: '#000', title: 'Color title' });
      const height = (49 - 33) + (this.colors.length * 33);
      // this.draggableMinHeight = height;
      this.draggableHeight = height;
    },
    remove_color() {
      if (this.colors.length === 1) return;
      const lastIndex = this.colors.length - 1;
      if (lastIndex === this.currentIndexColor) this.currentIndexColor = lastIndex - 1;

      countriesIds.forEach((country) => {
        if (this.countries[country] === lastIndex) {
          this.countries[country] = 0;
          document.getElementById(country).style.fill = this.colors[0].hex;
        }
      });
      this.colors.pop();
      const height = (49 - 33) + (this.colors.length * 33);
      this.draggableHeight = height;
    },
    colorsUpdated() {
      countriesIds.forEach((country) => {
        if (this.countries[country] === this.currentIndexColor) {
          document.getElementById(country).style.fill = this.colors[this.currentIndexColor].hex;
        }
      });
    },
    coloring(e) {
      if (e.target.tagName === 'path') {
        this.countries[e.target.id] = this.currentIndexColor;
        e.target.style.fill = this.colors[this.countries[e.target.id]].hex;
      }
    },
    saveImage(resolution = 1920) {
      const workplace = document.getElementById('workplace');
      const scale = resolution / workplace.offsetWidth;
      const tempCurrentIndexColor = this.currentIndexColor;
      const options = { scale };
      this.draggableActive = false;

      html2canvas(workplace, options)
        .then((canvas) => {
          const canWidth = canvas.width;
          const canHeight = canvas.height;
          const filename = 'map';

          Canvas2Image.saveAsPNG(canvas, canWidth, canHeight, filename);
        });
    },
    resize(newRect) {
      this.draggableWidth = newRect.width;
      this.draggableHeight = newRect.height;
    },
    drag(newRect) {
      this.draggableTop = newRect.top;
      this.draggableLeft = newRect.left;
    },
    mouseOverPalette(bool) {
      this.draggable = bool;
    },
    mouseOverControll(bool) {
      this.draggableControll = bool;
    },
  },
  beforeMount() {
    countriesIds.forEach((id) => {
      this.countries[id] = this.currentIndexColor;
    });
  },
};

</script>


<style>
/*#3f86c8
#fcfdf8
#f2f3f7*/
ul, ul li {
  text-align: left;
  list-style: none;
  padding: 0;
  margin: 0;
}

[contentEditable=true]:empty:not(:focus):before{
  content:attr(data-placeholder)
}

.btn-reset {
  margin: 0;
  padding: 0;
  border: none;
  font: inherit;
  color: inherit;
  background: none;
  cursor: pointer;
}
.btn-reset::-moz-focus-inner {padding: 0;border: none;}
.btn-reset:-moz-focusring {outline: 1px dotted;}

body {
  margin: 0;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background: #ececec;
  padding: 5px;
  box-sizing: border-box;
  min-height: 100vh;
}

#workplace {
  background: white;
  width: 1200px;
  margin: auto;
  position: relative;
  box-shadow: 0 0px 7px 0px #0000003d;
}

#colors {
  position: absolute;
  border: solid #cecece 3px;
  background: #f7f7f791;
  width: 100%;
  box-sizing: border-box;
  cursor: move;
  padding: 5px;
}
#colors:hover {
  border: solid #dededea1 3px;
  background: #dededea1;
}
.title {
  border: none;
  font-size: 24px;
  text-align: center;
  width: 480px;
  outline: none;
  margin: auto;
  color: black;
  box-sizing: border-box;
  padding: 10px;
}

.colors-picker {
  position: absolute;
  cursor: default;
  z-index: 2;
}

.color {
  height: 27px;
  padding: 3px;
}
.color > .content {
  display: inline-block;
  box-sizing: border-box;
  cursor: default;
  width: 100%;
}
.color span {
  width: 25px;
  height: 25px;
  display: inline-block;
  vertical-align: middle;
  z-index: 1;
}
.color input {
  background: none;
  border: none;
  outline: none;
  width: calc(100% - 30px);
  padding: 2px;
  box-sizing: border-box;
}
.selected {
  border: 2px solid #505050;
}
.vdr {
  position: absolute;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
}
.handle-ml, .handle-mr {
  top: 50%;
  cursor: ew-resize;
}
.handle-ml { left: -4px; margin-top: -4px; }
.handle-mr { right: -4px; margin-top: -4px; }

.handle {
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  position: absolute;
  font-size: 1px;
  background: #fff;
  border: 1px solid #6c6c6c;
  -webkit-box-shadow: 0 0 2px #bbb;
  box-shadow: 0 0 2px #bbb;
  width: 8px;
  height: 8px;
  z-index: 1;
}

#map {
  margin-left: 120px;
}

#made {
  position: absolute;
  bottom: 15px;
  width: 100%;
}
#made input {
  border: none;
  text-align: center;
  color: #2c3e50;
}

#controll {
  position: fixed;
  cursor: move;
  background: #3f86c8;
  padding: 10px;
  border-radius: 3px;
}
#controll * { cursor: pointer; }

.btn-add-colors {
  outline: none;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  background: #fcfdf8;
  text-align: center;
  vertical-align: middle;
  color: #3f86c8;
  font-weight: bold;
}
.btn-add-colors:hover, .btn-download:hover {background: #f2f3f7;}

.btn-download {
  color: #3f86c8;
  padding: 5px;
  background: #fcfdf8;
  font-weight: bold;
}

</style>
