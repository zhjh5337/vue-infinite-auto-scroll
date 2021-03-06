<template>
    <div class="infinite-warp" :class="wrapClass">
        <ul class="scroller" :class="scrollerClass">
            <li v-for="item in listData">
                <slot :item="item"></slot>
            </li>
        </ul>
    </div>
</template>
<script type="text/javascript">
export default {
    name: 'scroller',
    data() {
        return {
            listData: [],
            copyData: [],
            dataLength: 0,
            store: [],
            stroeLastIndex: -1
        }
    },
    props: {
        data: {
            type: Array,
            default: () => {
                return []
            }
        },
        //force:默认为false（如果列表总高度小于容器将不进行循环滚动），当为true时，不论列表总高度为多少都将进行循环滚动
        //speed:滚动速度
        //once:是否只滚动一次
        //newFirst:新增加的数据是否优先显示，此参数会使轮播的顺序和加入的顺序不一致
        option: {
            type: Object,
            default: () => {
              return {}
            }
        }
    },
    created(){
        //页面可能同时存在多个该组件
        window.scrollerCount = window.scrollerCount || 0;
        window.scrollerCount++;
        this.scrollerClass = 'scroller'+window.scrollerCount;
        this.wrapClass = 'infinite-warp'+window.scrollerCount;
        this.copyData = this.data.slice(0,this.data.length);
        this.dataLength = this.data.length;
        this.force = this.option.force || false;
        this.speed = this.option.speed || 1;
        this.once = this.option.once || false;
        this.newFirst = this.option.newFirst || false;
    },
    mounted(){
        this.scroll();
    },
    methods:{
        //开始滚动
        scroll(){
            let self = this;
            let wrap = document.querySelector('.'+self.wrapClass);
            let scroller = document.querySelector('.'+self.scrollerClass);
            if(!scroller)
                return;
            let translateY = self.getComputedTranslateY(scroller);
            let li = scroller.querySelectorAll('li');
            if(scroller.scrollHeight > Math.abs(translateY)+wrap.clientHeight){
                scroller.style = 'transform: translate3d(0px, '+(translateY-self.speed)+'px, 0px);';
            }else{ //只有列表总高度超过容器高度或者force参数为false时才循环无限滚动
                let tmp = [];
                let length = 0;
                tmp = self.copyData.splice(0,1);
                length = tmp.length;
                if(self.newFirst){
                    //新数据添加到缓存对象的头部（将第一个被取出）
                    self.store = tmp.concat(self.store);
                }else if(tmp.length){
                    //新数据添加到缓存对象的尾部（等待缓存对象中前面的数据展示完才会被取出）
                    self.store.splice(0,0,tmp[0]);
                }
                //一轮滚动还没结束（还有新数据未展示完）
                if(length){
                    //从缓存对象中取出第一个数据用于展示
                    tmp = self.store.splice(0,1);
                    //取出后再添加到尾部，用于循环展示
                    self.store = self.store.concat(tmp);
                }
                //一轮滚动结束后且满足循环滚动条件则从缓存里取，以实现循环无限滚动
                if(!length && !self.once &&
                    (self.force || (!self.force && li.length && self.data.length*li[0].clientHeight > wrap.clientHeight))){
                    //从缓存对象中取出第一个
                    tmp = self.store.splice(0,1);
                    //取出后再添加到尾部，用于循环展示
                    self.store = self.store.concat(tmp);
                }
                //将取出的数据展示到页面
                tmp.forEach(function(item){
                    self.$set(self.listData,self.listData.length,item);
                });
            }
            //删除顶部无用的dom，防止浏览器卡顿
            if(scroller && li.length){
                let liHieght = li[0].clientHeight;
                let delLenght = (Math.abs(translateY)/liHieght)>>0;
                if(delLenght>5){
                    self.listData.splice(0,delLenght);
                    scroller.style = 'transform: translate3d(0px, '+(translateY%liHieght)+'px, 0px);';
                }
            }
            window.requestAnimationFrame(function(){
                self.scroll();
            })
        },
        //获取计算后的translateY
        getComputedTranslateY: function(dom) {
            var startY = 0;
            var style = window.getComputedStyle ? window.getComputedStyle(dom, null) : null || dom.currentStyle;
            var matrix = style['transform'];
            if (matrix && matrix != 'none') {
                startY = Number(matrix.replace(/matrix\(|\)/g, '').split(',')[5]);
            }
            return startY;
        }
    },
    watch:{
        data: { 
            handler: function (val, oldVal) {  
                if(val.length>this.dataLength){
                    this.copyData = this.copyData.concat(val.slice(this.dataLength,val.length));
                }else if(val.length==0){
                    this.store = [];
                    this.copyData = [];
                    this.listData = [];
                }
                this.dataLength = val.length;
            },  
            deep: true
        }
    }
}

</script>
<style lang="scss">
    ul,li{
        margin: 0;
        padding: 0;
        list-style: none;
    }
    .infinite-warp{
        overflow: hidden;
    }
</style>
