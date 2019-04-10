<template>
         <div>
            <div class="ruler-heder" ref="rulerHeade">
                <span class="title">{{title}}:</span>
                <div class="ageBox">
                    <div class="numBox">
                        <span v-show="onff">{{initNum}}</span>
                        <span v-show="!onff">{{moveBgMark}}</span>
                        <span class="age">{{company}}</span>
                    </div> 
                </div>
            </div>
            <div class="ruler-scale-box">
                <ul 
                    ref="ul"
                    @touchstart='touchstart'
                    @touchmove='touchmove' 
                    @touchend='touchend'
                    class="ul"
                    data-offset="0">
                   <li v-show="index!=0" v-for="(item,index) in dataArr" :key="index" 
                   :class="{noMargin:index==0,longBg:item%10==0,addHeight:item%10==0,addBgc:item==moveBgMark,defualtBg:item==defaultMarkBg}"
                    class="li">
                       <span v-if="item%10==0">{{item}}</span>
                   </li>
                </ul>
            </div>
    </div>  
</template>
<script>
export default {
     data(){
         return{
            onff:true,
            screenW:null,  //屏幕的宽
            ulW:null,     //尺子的总宽
            initNum:null,  //初始化的Aage
            defaultMarkBg:null,
            liALLWidth:null, //一个li占的宽度
            slideX:null,     //每次移动更新的left
            ulLeft:null, //
            dataArr:[],
            ul:null, //缓存尺子
            ulmoveLeft:0,
            MarkulLeft:null,
            lens:0,
            liWidth:null,
            moveBgMark:0
         }
     },
     props:['title','initData','minValue','maxValue','company'],
     mounted(){
         console.log(this.$refs.rulerHeade.offsetWidth)
         if(this.initData<this.minValue){
            this.initNum = this.minValue
              this.defaultMarkBg = this.minValue
         }else if(this.initData>this.maxValue){
            this.initNum = this.maxValue
            this.defaultMarkBg = this.maxValue
         }else{
            this.initNum = this.initData
            this.defaultMarkBg = this.initData
         }
        for(var i=this.minValue;i<this.maxValue+1;i++){
            this.dataArr.push(i)
        }
        this.ul = this.$refs.ul
        this.$nextTick(()=>{
            var li = document.querySelectorAll('.li')[1]
         
            //屏幕的宽度
            this.screenW = this.$refs.rulerHeade.offsetWidth
            console.log('屏幕的长度',this.screenW)
            //获取每个li的宽度
            var liWidthPx = this.getStyle(li,'width')
            this.liWidth = Number(liWidthPx.substr(0,liWidthPx.length-2)) 
            //获取每个li的margin-left
            var marginLeftPx = this.getStyle(li,'marginLeft')
            var marginLeft = Number(marginLeftPx.substr(0,marginLeftPx.length-2)) 
            //一个li总占位宽度
            this.liALLWidth = marginLeft+this.liWidth
            //动态设置尺子的总宽
            this.ul.style.width = this.liALLWidth*(this.maxValue-this.minValue)+'px'
            this.ulW = this.ul.offsetWidth
            //初始化尺子偏移
            this.lens = -this.initNum*(this.liALLWidth)+this.screenW/2+this.liALLWidth*this.minValue
            this.ul.style.left = this.lens+'px'
            //将尺子初始的偏移量缓存
            this.ulmoveLeft = this.lens
        })
       
     },
     methods:{
        //获取元素的样式
        getStyle(obj,attr){ 
            if(obj.currentStyle){ 
                return obj.currentStyle[attr]; 
            } 
            else{ 
                return document.defaultView.getComputedStyle(obj,null)[attr]; 
            } 
        },
       
         touchstart(ev){
            let ulLeftPx =  this.getStyle(this.ul,'left')
            //按下手指获取尺子的left值
            this.ulLeft = Number(ulLeftPx.substr(0,ulLeftPx.length-2)) 
            //手指按下的坐标
            this.startX= ev.touches[0].clientX;   
         },
         touchmove(ev){
            this.onff =false
            this.defaultMarkBg = 0
            //移动停下的坐标
            this.moveX = ev.touches[0].clientX;
            //根据偏移量，计算出需要设置尺子的新的cleft
            this.slideX = this.moveX- this.startX+this.ulLeft
            // 设置尺子left 动起来
            this.ul.style.left = this.slideX + 'px'
            //再次获取尺子的left值
            let ulLeftPx =  this.getStyle(this.ul,'left')
            //将尺子的left带px==>数字
            this.MarkulLeft = Number(ulLeftPx.substr(0,ulLeftPx.length-2)) 
            let endMoveX = (this.MarkulLeft-this.ulmoveLeft)/this.liALLWidth
            //对小数个li 四舍五入
            let converNum =  endMoveX.toFixed(0)
            if(converNum<=0){
                this.moveBgMark =Math.abs( converNum)+this.initNum
            }else{
                this.moveBgMark =this.initNum - Math.abs(converNum)
            }

            if(this.moveBgMark<=this.minValue){
                // this.moveBgMark = 0
                this.moveBgMark = this.minValue
            }else if(this.moveBgMark>=this.maxValue){
                    // this.moveBgMark = 100
                    this.moveBgMark =this.maxValue
            }else{
                    this.moveBgMark =   this.moveBgMark
            }

            //限制尺子的边界
            if(this.MarkulLeft>=this.screenW/2){
                this.ul.style.left = this.screenW/2 + 'px'
                  
            }else if(this.MarkulLeft<-this.ulW+this.screenW/2){
                this.ul.style.left = -this.ulW+this.screenW/2+ 'px'
                  
            }else{
                this.ul.style.left = this.slideX + 'px'
            }
         },
         touchend(ev){
            this.onff =true
             //手指抬起计算偏移多少个li
            let endMoveX = (this.MarkulLeft-this.ulmoveLeft)/this.liALLWidth
            //对小数个li 四舍五入
            let converNum =  endMoveX.toFixed(0)
            if(converNum<0){
                this.initNum = this.initNum+Math.abs(converNum)
            }else{
                this.initNum = this.initNum-converNum
            }
            let ulLeftPx =  this.getStyle(this.ul,'left')
            this.ulmoveLeft =Number(ulLeftPx.substr(0,ulLeftPx.length-2)) 
         
           if(this.ulmoveLeft>=this.screenW/2){
                this.ul.style.left = this.screenW/2 + 'px'
                this.initNum = this.minValue
            }else if(this.ulmoveLeft<=-this.ulW+this.screenW/2){
                this.ul.style.left = -this.ulW+this.screenW/2+ 'px'
                this.initNum = this.maxValue
            }else{
                 this.lens = -this.initNum*(this.liALLWidth)+this.screenW/2+this.liALLWidth*this.minValue
                this.ul.style.left = this.lens+'px'
            }
        }
     }
}
</script>
<style scoped lang='less'>
    .ruler-container {
        overflow: hidden;
        width: 100%;
        height: 2400px;
        border: 1px solid rgb(147, 184, 47);
    }
    .ruler-heder{
        width: 100%;
        height: 45px;
        position: relative;
        margin-bottom: 18px;
            .title{
                display: inline-block;
                position: absolute;
                left: 0;
                top: 50%;
                transform: translateY(-50%);
                font-size: 32px;
                font-weight:500;
                color: #333
            }
            .ageBox{
                font-size: 0;
                display: inline-block;
                height:56px;
                position: absolute;
                top: 0;
                left: 50%;
                transform: translate(-50%,0%);
                line-height: 60px;
                .numBox{
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    position: relative;
                    font-size: 40px;
                     height:56px;
                     line-height: 56px;
                    color:#1B9160;
                    .age{
                        position: absolute;
                        top: -10%;
                        left: 120%;
                        font-size: 24px;
                    }
                }
            
            }
    
    }
    .ruler-scale-box{
        width: 100%;
        height: 116px;
        position: relative;
        overflow: hidden;    
        background:linear-gradient(273deg,rgba(248,248,248,1) 0%,rgba(252,252,252,1) 50%,rgba(248,248,248,1) 100%);
        box-shadow:0px 0px 8px 0px rgba(0,0,0,0.1);
        border-radius:4px;
        margin-bottom: 40px;
        ul{
            position: absolute;
            top: 0;
            height: 88px;
            background: #FAFAFA;
            padding-top: 4px;
            .noMargin{
                margin-left: 0
            }
            .addHeight{
                height: 48px;
            }
            .longBg{
                background: #c6c6c6;
            }
            .addBgc{
                background: #1B9160
            }
            .defualtBg{
                 background:#1B9160
            }
          
            li{
                list-style: none;
                width: 4px;
                height: 30px;
                background: #DBDBDB;
                float: left;
                margin-left: 20px;
                position: relative;
                border-radius: 3px;
                span{
                    position: absolute;
                    bottom: 0;
                    right: 0;
                    transform: translate(50%,120%);
                    color: #c6c6c6;
                }
            }
        }
    }

</style>


