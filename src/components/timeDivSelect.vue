<template>
  <div>
    <div class="timecontainer" id="timecontainer" @mousedown="mousedownfn">
     <div class="selectall"> <input type="checkbox" id="selectAllid" @click="selectAllornot"><label for="selectAllid" style="cursor:pointer;padding-left:5px;">全选</label></div>
     <div class="righttips">
       <div class="tiplist">
         <span class="current"></span>投放
       </div>
        <div class="tiplist">
         <span></span>不投放
       </div>
     </div>
     <div class="swrap">
      <div class="leftweek">
        <div class="weekname" v-for="(item,index) in weekName" :key="index">{{item}}</div>
      </div>
      <div class="mainbox">
          <div class="boxlist"><div class="list" :title="`${timeSetp}小时`" v-for="(item,index) in timeList" :class="{selected:item==1}" @click="setcurrent(item,index)" :key="index"></div></div>
        <div class="bottomtime">
          <div class="timeList" v-for="(item,index) in timeName" :key="index">{{item}}</div>
        </div>
      </div>
     </div>

    </div>

  </div>
</template>
<script>
export default {
  data() {
    return {
      weekName: ['星期一', '星期二', '星期三', '星期四', '星期五', '星期六', '星期日'],
      timeName: ['00:00~06:00', '06:00~12:00', '12:00~18:00', '18:00~24:00'],
      timeList: this.timeString.split(''),
      isSelect: true,
      selectBoxDashed: null,
      startX: null,
      startY: null,
      initx: null,
      scrollX:null,
      scrollY:null,
      inity: null
    }
  },
  props: {
    timeSetp: {
      type: Number,
      default() {
        return 0.5
      }
    },
    timeString: {
      type: String,
      default() {
        let _timeArray = []
        for (let i = 0; i < (24 / this.timeSetp) * 7; i++) {
          _timeArray.push('0')
        }
        return _timeArray.join('')
      }
    }
  },
  model: {
    prop: 'timeString',
    event: 'triggertime'
  },
  mounted () {
  },
  methods: {
    clearBubble(e) {
      if (e.stopPropagation) {
        e.stopPropagation()
      } else {
        e.cancelBubble = true
      }
      if (e.preventDefault) {
        e.preventDefault()
      } else {
        e.returnValue = false
      }
    },
    mousedownfn(e) {
      //  创建选框节点
      this.selectBoxDashed = document.createElement('div')
      this.selectBoxDashed.className = 'select-box-dashed'

      document.body.appendChild(this.selectBoxDashed)
      this.scrollX = document.documentElement.scrollLeft || document.body.scrollLeft
      this.scrollY = document.documentElement.scrollTop || document.body.scrollTop
      //  设置选框的初始位置
      this.startX = e.x + this.scrollX || e.clientX + this.scrollX
      this.startY = e.y + this.scrollY || e.clientY + this.scrollY
      this.selectBoxDashed.style.cssText = `left:${this.startX}px;top:${this.startY}px`
      //  清除事件冒泡、捕获
      this.clearBubble(e)
      document.getElementById('timecontainer').addEventListener('mousemove', this.mousemovefn)
      document.body.addEventListener('mouseup', this.mouseUpfn)
    },
    mousemovefn(e) {
      //  设置选框可见
      this.selectBoxDashed.style.display = 'block'
      //  根据鼠标移动，设置选框的位置、宽高
      this.initx = e.x + this.scrollX || e.clientX + this.scrollX
      this.inity = e.y + this.scrollY || e.clientY + this.scrollY
      //  暂存选框的位置及宽高，用于将 select-item 选中
      this.left = Math.min(this.initx, this.startX)
      this.top = Math.min(this.inity, this.startY)
      this.width = Math.abs(this.initx - this.startX)
      this.height = Math.abs(this.inity - this.startY)
      this.selectBoxDashed.style.left = `${this.left}px`
      this.selectBoxDashed.style.top = `${this.top}px`
      this.selectBoxDashed.style.width = `${this.width}px`
      this.selectBoxDashed.style.height = `${this.height}px`
      let fileDivs = document.getElementsByClassName('list')
      for (let i = 0; i < fileDivs.length; i++) {
        let itemX_pos = fileDivs[i].offsetWidth + fileDivs[i].offsetLeft
        let itemY_pos = fileDivs[i].offsetHeight + fileDivs[i].offsetTop
        let condition1 = itemX_pos > this.left
        let condition2 = itemY_pos > this.top
        let condition3 = fileDivs[i].offsetLeft < (this.left + this.width)
        let condition4 = fileDivs[i].offsetTop < (this.top + this.height)
        if (condition1 && condition2 && condition3 && condition4) {
          fileDivs[i].classList.add('temp-selected')
        } else {
          fileDivs[i].classList.remove('temp-selected')
        }
      }
      this.clearBubble(e)
    },
    mouseUpfn(e) {
      document.getElementById('timecontainer').removeEventListener('mousemove', this.mousemovefn)
      let selectDom = document.getElementsByClassName('temp-selected');
      [].slice.call(selectDom).forEach(item => {
        if (item.classList.contains('selected')) {
          item.classList.remove('selected')
        } else {
          item.classList.add('selected')
        }
        item.classList.remove('temp-selected')
      })
      if (this.selectBoxDashed) {
        try {
          this.selectBoxDashed.parentNode.removeChild(this.selectBoxDashed)
        } catch (err) {
          // console.log(err)
        }
      }
      let fileDivs = document.getElementsByClassName('list')
      for (let i = 0; i < fileDivs.length; i++) {
        if (fileDivs[i].classList.contains('selected')) {
          this.timeList[i] = '1'
        } else {
          this.timeList[i] = '0'
        }
      }
      this.$emit('triggertime', this.timeList.join(''))
      this.clearBubble(e)
    },
    setcurrent(item, index) {
      if (item == 0) {
        this.timeList.splice(index, 1, '1')
      } else {
        this.timeList.splice(index, 1, '0')
      }
    },
    selectAllornot(e) {
      this.timeList.forEach((item, index) => {
        this.timeList.splice(index, 1, Number(e.target.checked))
      })
      this.$emit('triggertime', this.timeList.join(''))
    }
  }

}
</script>
<style  scoped>
.timecontainer{width:780px;overflow: hidden;padding-top:30px;}
.swrap{width:780px;overflow: hidden;}
.leftweek{display: inline-block;width:50px;vertical-align: top;margin-top:5px;}
.leftweek .weekname{display: inline-block;width:50px;height:29px;line-height:29px;text-align: center;font-size:12px;}
.mainbox{display: inline-block;vertical-align: middle;padding: 10px 0px 2px 10px; width: 710px;overflow: hidden; background-color: #eee; }
.boxlist{overflow: hidden;}
.boxlist .list{float:left;height: 20px;width:10px;border: 1px solid #999; border-radius: 2px;margin: 0 1px 7px; cursor: pointer;background-color: #fff;}
.boxlist .list:nth-of-type(12n){margin-right:10px;}
.boxlist .list.selected{background-color:#0099ff;border-color:#0099ff}
.selectall{float:left;font-size:12px;margin-left:50px;}
.righttips{font-size:12px;float: right;}
.righttips .tiplist{display: inline-block;margin-right:10px;}
.righttips .tiplist span{display: inline-block;width:20px;height:10px;border:1px solid #999;margin-right:3px;}
.righttips .tiplist span.current{ background: #43a9ed;border-color: #43a9ed;}
.timeList{display: inline-block;vertical-align: middle;font-size:12px;width:175px;text-align: center}
</style>
<style>
.select-box-dashed{position: absolute;display: none;width: 0px;height: 0px;
            padding: 0px; margin: 0px;border: 1px dashed #0099ff;
            background-color: #c3d5ed;opacity: 0.5;filter: alpha(opacity=50);font-size: 0px;z-index: 99999;
        }
</style>

