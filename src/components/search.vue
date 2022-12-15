<template>
  <div class="search">
    <el-form :inline="true" @submit.native.prevent>
        <el-form-item label="搜索">
            <el-input v-model="form.name" placeholder="请输入歌曲/歌手" @keydown.enter.native="onSubmit"></el-input>
        </el-form-item>
        <el-form-item>
            <el-button type="primary" @click="onSubmit">搜索</el-button>
        </el-form-item>
    </el-form>
     <el-table
     height="400"
     :empty-text='control.empty'
      :data="tableData"
      style="width: 100%">
        <el-table-column
            prop="name"
            label="歌名">
        </el-table-column>
        <el-table-column
            prop="ator"
            label="歌手">
        </el-table-column>
        <el-table-column
            label="播放">
            <template slot-scope="scope">
                <el-button @click="bofangList(scope.row)">播放</el-button>
            </template>
        </el-table-column>
    </el-table>
    <el-pagination
        layout="next, pager, prev"
        :page-size='30'
        :total="300"
        @current-change='change'
        v-if="tableData.length || control.isPage">
  </el-pagination>
    <audio :src="control.src" autoplay ref="audio" @timeupdate="timeupdate(control.mp3)"></audio>
    <h1>{{control.lrcz}}</h1>
    <div class="bofangqi">
        <div class="introduce">
            <div class="img">
                <img :src="control.imgs" alt="" width="64" height="64" v-if='control.imgs'>
                <img src="../assets/1.jpg" alt="" width="64" height="64" v-else>
            </div>
            <div class="music">
                <h4>{{control.name?control.name:'暂无歌曲'}}</h4>
                <span>{{control.ators?control.ators:'暂无'}}</span>
            </div>
        </div>
        <div class="controlKey">
            <div class="key">
                <el-tooltip class="item" effect="dark" :content="`上一首：${control.preSong}`" placement="left">
                    <el-button :style="{border:'none',cursor:control.ispre?'pointer':'not-allowed'}" @click="previousSong">
                        <i class="el-icon-arrow-left"></i>
                    </el-button>
                </el-tooltip>
                <el-button style="border:none" @click="zanting">
                    <i class="el-icon-video-pause" v-if='(this.control.current<this.control.allTime &&control.bofangmc) || isNaN(this.control.allTime)'></i>
                    <i class="el-icon-video-play" v-else-if="this.control.current>=this.control.allTime || !control.bofangmc"></i>
                </el-button>
                <el-tooltip class="item" effect="dark" :content="`下一首：${control.nextSong}`" placement="right">
                    <el-button :style="{border:'none',cursor:control.isnext?'pointer':'not-allowed'}" @click="NextSong">
                        <i class="el-icon-arrow-right"></i>
                    </el-button>
                </el-tooltip>
            </div>
            <div class="progress">
                <el-slider v-model="control.current" :show-tooltip="false" :max="control.allTime" style="width:300px" @change="barpro"></el-slider>
                <span>{{current}}/{{allTime}}</span>
            </div>
        </div>
        <div class="funcType">
            <el-button class="mute" @click="ismutes">
                <i class="el-icon-microphone" v-if='!control.ismute'></i>
                <i class="el-icon-turn-off-microphone" v-else></i>
            </el-button>
            <div class="mutepros">
                <el-slider :value="control.volume" style="width:100px" @input="handerinput"></el-slider>
            </div>
        </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
    name:"search",
    data() {
        return {
            form:{
                name:'',
            },
            name:'',
            tableData:[],
            control:{
                // 歌曲src
                src:'',
                // 翻页数控制
                 offset:0,
                //  列表提示信息
                 empty:'暂无数据',
                 // 翻页控制
                isPage:false,
                 // 播放控制
                bofangmc:true,
                 // 歌词
                lrcz:'',
                // 当前歌曲词id
                lrcid:'',
                // 图片
                imgs:'',
                // 歌手
                ators:'',
                // 歌名
                name:'',
                // 静音
                ismute:false,
                // 当前时间(秒)
                current:0,
                // 总时间（秒）
                allTime:0,
                // 判断按下和抬起
                isDrop:false,
                // 上一首歌曲
                preSong:'暂无',
                // 控制上一首
                ispre:false,
                // 下一首歌曲
                nextSong:'暂无',
                // 控制下一首
                isnext:false,
                // 音量控制
                volume:20,
                // 歌词存储
                mp3:''
            },
        };
    },
    methods:{
        // 搜索音乐
        onSubmit(){
            this.name = this.form.name? this.form.name:this.name
            this.tableData = []


            axios.get(`http://pansida.cn:3000/cloudsearch?keywords=${this.name}&offset=${String(this.control.offset)}`).then(({data})=>{
                var songs = data.result.songs
                songs.forEach(item=>{
                    this.tableData.push({
                        // 歌名
                        name:item.name,
                        // 歌手
                        ator:item.ar[0].name,
                        // id
                        id:item.id,
                        // 图片url
                        imgurl:item.al.picUrl,
                    })
                })
            })
            this.control.empty = '加载中...'
            this.control.isPage = true
            this.form.name = ''
        },
        // 播放音乐
        bofangList(val){
            // 歌手
            this.control.name = val.name
            // 歌手
            this.control.ators = val.ator
            // 添加图片
            this.control.imgs = val.imgurl
            // 添加歌词
            this.control.lrcz = ''
            this.control.lrcid = val.id
            // 歌曲获取
            axios.get(`http://pansida.cn:3000/song/url?id=${val.id}`).then(({data})=>{
                const url = data.data[0].url
                this.control.src = url
                this.control.lrcid = val.id
            })
            // 歌词获取
            axios.get(`http://pansida.cn:3000/lyric?id=${this.control.lrcid}`).then(({data})=>{
                this.control.mp3 = data.lrc.lyric
            })
            // 上一首歌显示
            const preindex = this.tableData.findIndex(item=>item === val)
            const predata = this.tableData[preindex-1]
            if(predata){
                this.control.preSong = predata.name
                this.control.ispre = true
            }
            else{
                this.control.preSong = '暂无'
                this.control.ispre = false
            }
            // 下一首歌显示
            const nextindex = this.tableData.findIndex(item=>item === val)
            const nextdata = this.tableData[nextindex+1]
            if(nextdata){
                this.control.nextSong = nextdata.name
                this.control.isnext = true
            }
            else{
                this.control.nextSong = '暂无'
                this.control.isnext = false
            }

        },
        // 暂停音乐
        zanting(){
            if(this.control.src){
                if(this.control.bofangmc){
                    this.$refs.audio.pause()
                    this.control.bofangmc = false
                }else{
                    this.$refs.audio.play()
                    this.control.bofangmc = true
                }
            }
        },
        // 上一首
        previousSong(){
            if(this.control.ispre){
                const index = this.tableData.findIndex(item=>{
                    return item.id === this.control.lrcid
                })
                const data = this.tableData[index-1]
                this.bofangList(data)
            }
        },
        // 下一首
        NextSong(){
            if(this.control.isnext){
                const index = this.tableData.findIndex(item=>{
                    return item.id === this.control.lrcid
                })
                const data = this.tableData[index+1]
                this.bofangList(data)
            }
        },
        // 出现分页器
        change(data){
            this.control.offset = (data-1)*30
            this.onSubmit()
        },
        // 歌词处理
        timeupdate(mp3){
                const { currentTime ,duration} = this.$refs.audio
                this.control.current = currentTime
                this.control.allTime = duration
                let minutes = parseInt(currentTime/60)
                let seconds = parseInt(currentTime%60)
                let time = (minutes<10?'0'+minutes:minutes)+':'+ (seconds<10?'0'+seconds:seconds)
                const reg = new RegExp('\\['+time+'\\.\\d{2,3}\\](.+)')
                let lrc = reg.exec(mp3)
                lrc = lrc?lrc[1].replaceAll("(\n|\r|(\r\n)|(\u0085)|(\u2028)|(\u2029))", ""): ''
                this.control.lrcz = lrc? lrc:this.control.lrcz
        },
        // 是否静音
        ismutes(){
            if(!this.control.ismute){
                this.control.ismute = true
                this.$refs.audio.muted = true
            }
            else{
                this.control.ismute = false
                this.$refs.audio.muted =false
            }
        },
        // 音量控制
        handerinput(value){
            this.control.volume = value
            if(this.control.volume === 0){
                this.$refs.audio.volume = this.control.volume / 100
                this.control.ismute = true
            }
            if(this.control.volume > 0){
                this.$refs.audio.volume = this.control.volume / 100
                this.control.ismute = false
            }
        },
        // 进度条改变时间
        barpro(news){
            this.$refs.audio.currentTime = news
        }     
    },
    computed:{
        // 当前时间
        current(){
            let minutes = parseInt(this.control.current/60)
            let seconds = parseInt(this.control.current%60)
            let time = (minutes<10?'0'+minutes:minutes)+':'+ (seconds<10?'0'+seconds:seconds)
            return time
            },
        // 总时间
        allTime(){
            if(isNaN(this.control.allTime)){
                return '00:00'
            }
            else{
                let minutes = parseInt(this.control.allTime/60)
                let seconds = parseInt(this.control.allTime%60)
                let time = (minutes<10?'0'+minutes:minutes)+':'+ (seconds<10?'0'+seconds:seconds)
                return time
            }
        }, 
    },
    mounted(){
        this.control.audio = this.$refs.audio
    }
}
</script>

<style lang='less' scoped>
h1{
    color:green;
    text-align: center;
}
.el-pagination{
    display: flex;
    flex-direction: row-reverse;
}
.bofangqi{
    min-width: 1546px;
    background: #fff;
    border-top: 1px #666 dashed;
    position: fixed;
    left: 0;
    top: 620px;
    width: 100%;
    height: 100px;
    display: flex;
    padding-left: 20px;
    padding-right: 20px;
    align-items: center;
    
    .introduce{
        width: 31%;
        display: flex;
        height: 64px;
        .img{
            img{border-radius: 10px;}
        }
        .music{
            margin-left: 20px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            h4{
                font-size: 18px;
                margin: 0;
            }
        }
    }
    .controlKey{
        width: 38%;
        .key{
            display: flex;
            justify-content: center;
            i{
                font-size: 30px;
            }
        }
        .progress{
            display: flex;
            justify-content: center;
            align-items: center;
            position: relative;
            margin-top: 5px;
            span{
                margin-left: 20px;
                color: #666;
                font-size: 12px;
            }
            .el-progress{
                position: relative;
                margin-left: 5px;
                margin-right: 5px;
                width: 300px;
            }
            .handle{
                cursor: pointer;
                position: absolute;
                left:102px;
                top:22%;
                width: 10px;
                height: 10px;
                background: #409eff;
                border-radius: 50%;
            }
        }
    }
    .funcType{
        margin-left: 10px;
        width: 31%;
        display: flex;
        .mute{
            border: none;
            i{
                font-size: 30px;
            }
        }
        .mutepros{
            display: flex;
            align-items: center;
            .mutepro{
            width: 100px;
            margin-left: 10px;
        }
        }
    }

}
</style>