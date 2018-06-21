<template>
    <div class="list" id="viewbox">
        <div v-for="(ele,idx) in renderData" :key="idx">
            <div v-for="(item,index) in ele.list" :key="index" class="item" :style="`height:${item}px;line-height:${item}px`">{{item}}</div>
        </div>
    </div>
</template>

<script>
//移除事件
const removeEvent = (element, eType,bol) => {
	if(!element.eventSave) return false;
	let handleType = eType.split('.')[0];
	let handleName = eType.split('.')[1] || 'all';
	if(!element.eventSave[handleName + handleType]) return false;
	let handle = element.eventSave[handleName + handleType];
    if(element.addEventListener){
		bol = bol == undefined ? false : bol;
        element.removeEventListener(handleType, handle, bol);
    }else if(element.attachEvent){
        element.detachEvent("on"+handleType, handle);
    }else{
        element["on"+handleType] = null;
    }
}
//綁定事件
const addEvent = (element, eType, handle, bol) => {
	if(!element.eventSave){
		element.eventSave = new Object;
	}
	let handleType = eType.split('.')[0];
	let handleName = eType.split('.')[1] || 'all';
	if(!handleType) return false;
	//如果原來有一樣的事件，就先解綁，然後給綁定新的事件
	if(element.eventSave[handleName + handleType] != undefined){
		removeEvent(element,eType);
	}
  element.eventSave[handleName + handleType] = handle;
	//如果支持addEventListener
	if (element.addEventListener) {
		bol = bol == undefined ? false : bol;
		element.addEventListener(handleType, handle, bol);
	} else if (element.attachEvent) {
		//如果支持attachEvent
		element.attachEvent("on" + handleType, handle);
	} else {
		//否则使用兼容的onclick绑定
		element["on" + handleType] = handle;
	}
}
//獲取滾動條的位置
const getScrollPosition = () => {
	if (window.pageYOffset != null) {
		return {
			left: window.pageXOffset,
			top: window.pageYOffset
		}
	} else if (document.compatMode == 'CSS1Compat') {
		return {
			left: document.documentElement.scrollLeft,
			top: document.documentElement.scrollTop
		}
	}
	return {
		left: document.body.scrollLeft,
		top: document.body.scrollTop
	}
}

export default {
  data() {
    return {
      group: 8,
      renderData: [],
      contentBoxHeight : 0,
      viewBoxHeight : document.documentElement.clientHeight,
    };
  },
  created() {
    this.insert();
    addEvent(window,'scroll.list',() => {
        let scrollTop = getScrollPosition().top;
        let contentHeight = this.contentBoxHeight;
        let windowHeight = this.viewBoxHeight;
        if(scrollTop + windowHeight >= contentHeight * 0.8){
            this.insert();
        }
    });
    addEvent(window,'resize.list',() => {
        this.viewBoxHeight = document.documentElement.clientHeight;
    });
  },
  destroyed(){
      removeEvent(window,'scroll.list');
      removeEvent(window,'resize.list');
  },
  methods: {
    //插入數據
    insert(){
        let data = this.mockData();
        this.insertData(data);
        this.$nextTick(() => {
            this.contentBoxHeight = document.documentElement.offsetHeight;
        });
    },
    //獲取mock數據
    mockData() {
      let arr = [];
      for (let i = 0; i < 100; i++) {
        arr.push(Math.floor(Math.random() * 140) + 50);
      }
      return arr;
    },
    //獲取插入隊列的下標
    getInsertIdx() {
      let list = this.renderData;
      let insertIdx = 0;
      let sum = list[insertIdx].sum;
      list.forEach((ele, idx) => {
        if (sum > ele.sum) {
          insertIdx = idx;
        }
      });
      return insertIdx;
    },
    //插入數據到隊列
    insertData(list) {
      let renderData = this.renderData;
      if (typeof renderData[0] != "object") {
        for (let i = 0; i < this.group; i++) {
          this.renderData.push({
            list: [],
            sum: 0
          });     
        }
      }

      list.forEach((ele, idx) => {
        let insertIdx = this.getInsertIdx();
        this.renderData[insertIdx].sum += ele;
        this.renderData[insertIdx].list.push(ele);
        console.log(JSON.stringify(this.renderData));
      });

    },
  }
};
</script>

<style>
.list{
    display: flex;
}
.item {
  width: 150px;
  border: 1px solid #ccc;
  margin: 10px;
  text-align: center;
}
</style>