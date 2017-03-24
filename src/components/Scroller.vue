<template>
  <div class="vscroller">
    <div class="vscroller__head">{{scrollerTitle}}</div>
    <div class="vscroller__body">
      <ul :style="{ webkitTransition: '-webkit-transform ' + transitionDuration / 1000 + 's ease-out', transition: 'transform ' + transitionDuration / 1000 + 's ease-out', webkitTransform: 'translate3d(0px, ' + currentTranslatedY + 'px, 0px)', transform: 'translate3d(0px, ' + currentTranslatedY + 'px, 0px)' }" ref="ul">
        <li v-for="item in itemList" :key="item.id" :data-val="item.value" :class="item.selected ? 'vselected' : ''">{{item.name}}</li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Scroller',
  props: {
    scrollerTitle: String,
    itemList: Array,
  },
  data() {
    return {
      selectedIndex: 0,
      selectedValue: 1,
      startPosY: 0,
      currentPosY: 0,
      startTime: 0,
      endTime: 0,
      lastTime: (new Date()).getTime(),
      transitionDuration: 0,
      lastPosY: 0,
      lastV: 0,
      startTranslatedY: 80,
      currentTranslatedY: 80,
      havedClicked: false,
      isMouseDown: false,
      totalHeight: 40,
    };
  },
  created() {
  },
  mounted() {
    let supportedTouch = false;
    this.initData();
    if ('ontouchstart' in window) {
      supportedTouch = true;
    }
    if (supportedTouch) {
      this.bindTouchEvents();
    } else {
      this.bindMouseEvents();
    }
    this.bindClickEvent();
  },
  computed: {
  },
  methods: {
    initData() {
      if (this.itemList.length > 0) {
        this.itemList.forEach((item, index) => {
          if (item.selected === true) {
            this.selectedValue = item.value;
            this.selectedIndex = index;
            return false;
          }
          return true;
        });
        this.startTranslatedY = 80 - (this.selectedIndex * 40);
        this.currentTranslatedY = this.startTranslatedY;
      } else {
        this.itemList = [{ value: 1, name: 'Please select..', selected: true }];
      }
      this.totalHeight = this.itemList.length * 40;
    },
    bindTouchEvents() {
      const el = this.$refs.ul;
      // bind events
      el.addEventListener('touchstart', (e) => {
        this.startPosY = e.changedTouches[0].pageY;
        this.currentPosY = this.startPosY;
        this.startTime = (new Date()).getTime();
        this.startTranslatedY = this.currentTranslatedY;
        this.lastV = 0;
        // console.log('touchstart!');
      }, false);

      el.addEventListener('touchmove', (e) => {
        e.preventDefault(); // prevent default scrolling event when touch moving
        this.lastV = (e.changedTouches[0].pageY - this.lastPosY)
         / ((new Date()).getTime() - this.lastTime);
        this.currentPosY = e.changedTouches[0].pageY;
        this.currentTranslatedY = (this.startTranslatedY + this.currentPosY) - this.startPosY;
        this.lastPosY = this.currentPosY;
        this.lastTime = (new Date()).getTime();
      }, false);

      el.addEventListener('touchend', () => {
        this.endTime = (new Date()).getTime();
        if (Math.abs(this.currentPosY - this.startPosY) > 5 && this.endTime - this.startTime > 50) {
          const v = this.lastV;
          const s = v > 0 ? (0.5 * (v ** 2)) / 0.001 : (-0.5 * (v ** 2)) / 0.001;
          const t = Math.abs(v) / 0.001;
          let currentTranslatedY = this.currentTranslatedY;
          currentTranslatedY += s;
          const residue = currentTranslatedY % 40;
          if (Math.abs(residue) >= 20) {
            if (residue < 0) {
              currentTranslatedY += ((40 + residue) * (-1));
            } else {
              currentTranslatedY += (40 - residue);
            }
          } else {
            currentTranslatedY -= residue;
          }
          if (currentTranslatedY > 80) {
            currentTranslatedY = 80;
          } else if (currentTranslatedY < (this.totalHeight - 120) * (-1)) {
            currentTranslatedY = (this.totalHeight - 120) * (-1);
          }
          const selectedIndex = Math.abs((currentTranslatedY - 80) / (-40));
          this.transitionDuration = t;
          this.currentTranslatedY = currentTranslatedY;
          setTimeout(() => {
            this.itemList[this.selectedIndex].selected = false;
            this.selectedIndex = selectedIndex;
            this.itemList[this.selectedIndex].selected = true;
            this.selectedValue = this.itemList[this.selectedIndex].value;
            this.havedClicked = false;
          }, t);
        } else {
          this.havedClicked = true;
        }
        this.startPosY = 0;
        this.currentPosY = 0;
        this.startTranslatedY = 0;
        this.startTime = 0;
        this.endTime = 0;
        this.lastPosY = 0;
        this.lastV = 0;
      }, false);
    },
    bindMouseEvents() {
      const el = this.$refs.ul;
      let mouseDown = null;
      let mouseMove = null;
      let mouseUp = null;
      let mouseLeave = null;
      let mouseWheel = null;

      mouseDown = (e) => { // mouse down event
        this.isMouseDown = true;
        this.startPosY = e.pageY;
        this.currentPosY = this.startPosY;
        this.startTime = (new Date()).getTime();
        this.startTranslatedY = this.currentTranslatedY;
        el.addEventListener('mousemove', mouseMove);
        el.addEventListener('mouseup', mouseUp);
        el.addEventListener('mouseleave', mouseLeave);
        // console.log('mouseDown!');
      };
      mouseMove = (e) => { // mouse move event
        if (this.isMouseDown) {
          e.preventDefault(); // prevent default selecting event when mouse moving
          this.lastV = (e.pageY - this.lastPosY)
           / ((new Date()).getTime() - this.lastTime);
          this.currentPosY = e.pageY;
          this.currentTranslatedY = (this.startTranslatedY + this.currentPosY) - this.startPosY;
          this.lastPosY = this.currentPosY;
          this.lastTime = (new Date()).getTime();
          this.havedClicked = false;
        }
      };
      mouseUp = () => { // mouse up event
        this.endTime = (new Date()).getTime();
        if (Math.abs(this.currentPosY - this.startPosY) > 5 && this.endTime - this.startTime > 20) {
          const v = this.lastV;
          const s = v > 0 ? (0.5 * (v ** 2)) / 0.001 : (-0.5 * (v ** 2)) / 0.001;
          const t = Math.abs(v) / 0.001;
          let currentTranslatedY = this.currentTranslatedY;
          currentTranslatedY += s;
          const residue = currentTranslatedY % 40;
          if (Math.abs(residue) >= 20) {
            if (residue < 0) {
              currentTranslatedY += ((40 + residue) * (-1));
            } else {
              currentTranslatedY += (40 - residue);
            }
          } else {
            currentTranslatedY -= residue;
          }
          if (currentTranslatedY > 80) {
            currentTranslatedY = 80;
          } else if (currentTranslatedY < (this.totalHeight - 120) * (-1)) {
            currentTranslatedY = (this.totalHeight - 120) * (-1);
          }
          const selectedIndex = Math.abs((currentTranslatedY - 80) / (-40));
          this.transitionDuration = t;
          this.currentTranslatedY = currentTranslatedY;
          setTimeout(() => {
            this.itemList[this.selectedIndex].selected = false;
            this.selectedIndex = selectedIndex;
            this.itemList[this.selectedIndex].selected = true;
            this.selectedValue = this.itemList[this.selectedIndex].value;
            this.havedClicked = false;
          }, t);
        } else {
          this.havedClicked = true;
        }
        this.startPosY = 0;
        this.currentPosY = 0;
        this.startTranslatedY = 0;
        this.startTime = 0;
        this.endTime = 0;
        this.lastPosY = 0;
        this.lastV = 0;
        this.isMouseDown = false;
        el.removeEventListener('mousemove', mouseMove);
        el.removeEventListener('mouseup', mouseUp);
        el.removeEventListener('mouseleave', mouseLeave);
        // console.log('mouseUp!');
      };
      mouseLeave = () => { // mouse leave event
        if (this.isMouseDown) {
          mouseUp();
          // console.log('mouseLeave!');
        }
      };
      mouseWheel = (e) => { // mouse wheel event
        this.startTranslatedY = this.currentTranslatedY;
        let currentTranslatedY = this.startTranslatedY + (e.deltaY * 0.5);
        const residue = currentTranslatedY % 40;
        if (Math.abs(residue) >= 20) {
          if (residue < 0) {
            currentTranslatedY += ((40 + residue) * (-1));
          } else {
            currentTranslatedY += (40 - residue);
          }
        } else {
          currentTranslatedY -= residue;
        }
        if (currentTranslatedY > 80) {
          currentTranslatedY = 80;
        } else if (currentTranslatedY < (this.totalHeight - 120) * (-1)) {
          currentTranslatedY = (this.totalHeight - 120) * (-1);
        }
        this.transitionDuration = 0.2;
        this.currentTranslatedY = currentTranslatedY;
        const selectedIndex = Math.abs((currentTranslatedY - 80) / (-40));
        setTimeout(() => {
          this.itemList[this.selectedIndex].selected = false;
          this.selectedIndex = selectedIndex;
          this.itemList[this.selectedIndex].selected = true;
          this.selectedValue = this.itemList[this.selectedIndex].value;
        }, this.transitionDuration);
        this.startTranslatedY = 0;
      };
      // bind events
      el.addEventListener('mousedown', mouseDown);
      el.addEventListener('wheel', mouseWheel);
    },
    bindClickEvent() {
      const el = this.$refs.ul;
      el.querySelectorAll('li').forEach(($li, index) => {
        $li.addEventListener('click', () => {
          if (this.havedClicked) {
            const itemPositionY = $li.offsetTop;
            const currentTranslatedY = 80 - itemPositionY;
            this.transitionDuration = 0;
            this.currentTranslatedY = currentTranslatedY;
            this.itemList[this.selectedIndex].selected = false;
            this.selectedIndex = index;
            this.itemList[this.selectedIndex].selected = true;
            this.selectedValue = this.itemList[this.selectedIndex].value;
            this.havedClicked = false;
          }
        });
        return true;
      });
    },
  },
  watch: {
    itemList: () => {
      this.totalHeight = this.itemList.length * 40;
    },
  },
};
</script>

<style>
.vscroller{
  font-family: arial,verdana,sans-serif;
  padding: .5em .25em;
  min-width: 40px;
  max-width: 100%;
  -webkit-box-flex: 1;
  -webkit-flex: 1 auto;
  -ms-flex: 1 auto;
  flex: 1 auto;
  -ms-touch-action: none;
  touch-action: none;  
}

.vscroller__head {
  line-height: 30px;
  height: 30px;
  text-align: center;
  white-space: nowrap;
  font-weight: bold;
  text-transform: upperscse;
  color: #4eccc4;
  width: 100%;
  font-size: 1.375em;
}

.vscroller__body {
  height: 200px;
  overflow: hidden;
  position: relative;
}
.vscroller__body:before {
  content: '';
  position: absolute;
  left: 0;
  top: 80px;
  bottom: auto;
  right: auto;
  height: 1px;
  width: 100%;
  background-color: #4eccc4;
  display: block;
  z-index: 1;
}
.vscroller__body:after {
  content: '';
  position: absolute;
  left: 0;
  bottom: 80px;
  right: auto;
  top: auto;
  height: 1px;
  width: 100%;
  background-color: #4eccc4;
  display: block;
  z-index: 1;
}
.vscroller__body ul {
  display: block;
  margin: 0;
  padding: 0;
  position: relative;
  z-index: 3;
  transform: translate3d(0px, 0px, 0px);
}
.vscroller__body li {
  display: block;
  height: 40px;
  line-height: 40px;
  cursor: pointer;
  text-align: center;
  padding: 0 5px;
  margin: 0;
  white-space: nowrap;
  vertical-align: bottom;
  width: 100%;
  color: #454545;
  position: relative;
  overflow: hidden;
  text-overflow: ellipsis;
}
.vscroller__body li.vselected {
  font-weight: 700;
}
</style>
