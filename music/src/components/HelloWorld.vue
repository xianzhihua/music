<template>
  <div class="Index" id="Index">
    <div class="bg" :style="{backgroundImage : 'url(' + music_pic + ')'}">
    </div>
    <div class="title text-center">    
      <h2>一站式音乐播放器</h2>
      <p><a href="http://www.zhihuas.com">由 www.zhihuas.com 提供技术</a></p>
    </div>
    <div class="search-div">
      <form @submit.prevent.stop="onSubmit()">
        <input type="text" name="keywords" v-model="keywords" class="search text-center" placeholder="例如：爱的供养" autocomplete="off">
        <div class="player-type">
          <label>
            <input type="radio" name="163" value="163" v-model="playerType" checked="">
            <span>网易云</span>
          </label>
          <label>
            <input type="radio" name="qq" value="qq" v-model="playerType">
            <span>QQ</span>
          </label>
        </div>
      </form>
    </div>
    <div class="playlist" v-if="search">
      <div class="music" v-if="loading">
          <i></i>
          <i></i>
          <i></i>
          <i></i>
          <i></i>
          <i></i>
      </div>
      <div v-else>
        <div class="player-controller">
          <audio id="audio" class="bgm" type="audio/ogg" :src="music_url" :autoplay="music_play"></audio>
          <div class="music-pic">
            <div :style="{backgroundImage : 'url(' + music_pic + ')'}"></div>
          </div>
          <div class="music-info">
            <h3>{{music_title}}</h3>
            <div class="play-btn">
              <div class="prev" @click="getSong(music_index - 1,playerType)"></div>
              <div :class="!music_play ? 'play' : 'pause'" @click="stopMusic()"></div>
              <div class="next" @click="getSong(music_index + 1,playerType)"></div>
            </div>
            <div id="lyric">
              <ul id="lyric_ul" :style="{marginTop: (- index + 2) * 20 + 'px' }">
                <li v-for="(lyrics,cindex) in lyrics" :class="cindex == index ? 'cw' : ''">{{lyrics}}</li>
              </ul>
            </div>
            <!-- 进度条 -->
            <div class="progress" id="progress" @click="progressClick($event,true)" @mousedown="progressMove($event)">
              <div class="progressBar" id="progressBar"></div>
              <div class="progressDot" id="progressDot"></div>
              <div class="progressTime text-center">{{music_timeStr}}</div>
            </div>
            <!-- 音量 -->
            <div class="music_voice_ground">
              <div class="music_voice_icon" @click="musicVoiceSet()">
                <svg xmlns:xlink="http://www.w3.org/1999/xlink" height="100%" version="1.1" viewBox="0 0 28 32" width="100%" v-if="music_voice == 0">
                    <use xlink:href="#aplayer-volume-off"></use>
                    <path class="aplayer-fill" d="M13.728 6.272v19.456q0 0.448-0.352 0.8t-0.8 0.32-0.8-0.32l-5.952-5.952h-4.672q-0.48 0-0.8-0.352t-0.352-0.8v-6.848q0-0.48 0.352-0.8t0.8-0.352h4.672l5.952-5.952q0.32-0.32 0.8-0.32t0.8 0.32 0.352 0.8z" id="aplayer-volume-off"></path>
                </svg>
                <svg xmlns:xlink="http://www.w3.org/1999/xlink" height="100%" version="1.1" viewBox="0 0 28 32" width="100%" v-else="music_voice > 0">
                    <use xlink:href="#aplayer-volume-down"></use>
                    <path class="aplayer-fill" d="M13.728 6.272v19.456q0 0.448-0.352 0.8t-0.8 0.32-0.8-0.32l-5.952-5.952h-4.672q-0.48 0-0.8-0.352t-0.352-0.8v-6.848q0-0.48 0.352-0.8t0.8-0.352h4.672l5.952-5.952q0.32-0.32 0.8-0.32t0.8 0.32 0.352 0.8zM20.576 16q0 1.344-0.768 2.528t-2.016 1.664q-0.16 0.096-0.448 0.096-0.448 0-0.8-0.32t-0.32-0.832q0-0.384 0.192-0.64t0.544-0.448 0.608-0.384 0.512-0.64 0.192-1.024-0.192-1.024-0.512-0.64-0.608-0.384-0.544-0.448-0.192-0.64q0-0.48 0.32-0.832t0.8-0.32q0.288 0 0.448 0.096 1.248 0.48 2.016 1.664t0.768 2.528z" id="aplayer-volume-down"></path>
                </svg>
              </div>
              <div class="music_voice_progress" id="progressVoice" @click="voiceClick($event)">
                <div class="music_voice_progressBar" id="progressVoiceBar"></div>
              </div>
            </div>
          </div>
        </div>
        <div class="player-list">
          <ol v-if="playerType == '163'">
            <li :class="music_index == index ? 'active' : ''" v-for="(item,index) in playlist" @click="getSong(index,playerType)">
              <span class="border-left"></span>
              <span class="text-center">{{index + 1}}</span>
              <span>{{item.name}}</span>
              <span> <i v-for="ar in item.ar">{{ar.name}}</i> </span>
            </li>
            <p class="more text-center" v-show="songCount != 0 && songCount != playlist.length" @click="getPlayList(false,playerType)">加载更多</p>
          </ol>

          <ol v-else="playerType == 'qq'">
            <li :class="music_index == index ? 'active' : ''" v-for="(item,index) in playlist" @click="getSong(index,playerType)">
              <span class="border-left"></span>
              <span class="text-center">{{index + 1}}</span>
              <span>{{item.title}}</span>
              <span> <i v-for="ar in item.singer">{{ar.name}}</i> </span>
            </li>
            <p class="more text-center" v-show="songCount != 0 && songCount != playlist.length" @click="getPlayList(false,playerType)">加载更多</p>
          </ol>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data(){
    return {
      loading: true,
      search: false,
      isMove: false,      // 是否拖拽
      playerType: '163',  // 音乐播放器类型 163 qq
      keywords: '',       // 搜索关键字
      playlist: [],       // 歌曲列表
      music_url: '',      // 歌曲链接
      music_pic: 'https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1537285534425&di=88c84dcc0836a607e448212c62bebfaa&imgtype=0&src=http%3A%2F%2Fbpic.ooopic.com%2F15%2F47%2F11%2F15471198-274e37493e5532668820410d60ba20ce.jpg',      // 歌曲缩略图
      music_title: '',    // 歌曲标题
      times: [],          // 歌词时间
      lyrics: [],         // 歌词歌词
      index: -1,          // 歌词时间索引
      music_index: 0,     // 播放的第几首
      music_timeStr: '',  // 歌曲的总时间
      music_value: '',    // 进度条比例
      music_voice: 1,     // 音量大小
      music_play: true,   // 是否自动播放
      songCount: 0,       // 搜索到的歌曲数量
      limit: 30,          // 每次搜索多少条
      offset: 0,          // 163从第几条开始搜索
      page: 1             // qq
    }
  },
  methods: {
    onSubmit(){
      this.loading = true;
      this.search = true;
      this.playlist = [];
      this.offset = 0;
      this.page = 1;
      this.music_index = 0;
      this.getPlayList(true,this.playerType);
    },
    getPlayList(reset,playerType){
      var params = new URLSearchParams();
          params.append('playerType', this.playerType);
      switch(playerType){
        case '163' :
          params.append('s', this.keywords.replace(/\s+/g,""));
          params.append('limit', this.limit);
          params.append('offset', this.offset*this.limit);
          axios.post('http://localhost/music/public/index.php/api/api/getPlayList',params).then(res=>{
            for(var i in res.data.result.songs){
              this.playlist.push(res.data.result.songs[i]);
            }
            this.songCount = res.data.result.songCount;
            this.offset += 1;
            if(reset){
              this.getSong(this.music_index,playerType);
            }
          })
        break;
        case 'qq' :
          params.append('p', this.page);
          params.append('n', this.limit);
          params.append('w', this.keywords.replace(/\s+/g,""));
          axios.post('http://localhost/music/public/index.php/api/api/getPlayList',params).then(res=>{
            for(var i in res.data.data.song.list){
              this.playlist.push(res.data.data.song.list[i]);
            }
            this.songCount = res.data.data.song.totalnum;
            this.page += 1;
            if(reset){
              this.getSong(this.music_index,playerType);
            }
          })
        break;
      }
    },
    getSong(index,playerType){
      this.loading = false;
      this.times = [];
      this.lyrics = [];
      this.index = -1;
      if(index<0){
        index=this.playlist.length-1;
      }else if(index>this.playlist.length-1){
        index=0;
      }
      this.music_index = index;
      var params = new URLSearchParams();
      params.append('playerType', playerType);
      switch(playerType){
        case '163' :
          var ids = this.playlist[index].id;
          params.append('ids', ids);
          axios.post('http://localhost/music/public/index.php/api/api/getSong',params).then(res=>{
            this.music_url= res.data.data[0].url;
            this.music_pic= this.playlist[index].al.picUrl;
            this.music_title= this.playlist[index].name;
            this.endedMusic(playerType);
          });
          axios.post('http://localhost/music/public/index.php/api/api/getLyric',params).then(res=>{
            this.parseLyric(res.data.lrc.lyric);
          });
        break;
        case 'qq' :
          var ids = this.playlist[index].mid;
          params.append('ids', ids);
          axios.post('http://localhost/music/public/index.php/api/api/getSong',params).then(res=>{
            console.log(res)
            this.music_url=res.data;
            this.music_pic='https://y.gtimg.cn/music/photo_new/T002R300x300M000'+this.playlist[index].album.mid+'.jpg?max_age=2592000'
            this.music_title=this.playlist[index].title;
            this.endedMusic(playerType);
          })
        break;
      }
    },
    parseLyric(lyric){
      var _this = this;
      var array = lyric.split("\n");
      // [00:00.92]
      var timeReg = /\[(\d*:\d*\.\d*)\]/
      array.forEach(function(value,index){
        // 处理歌词
        var lrc = value.split("]")[1];
        // 排除空歌词
        if(lrc=='' || lrc==null) return true;
        _this.lyrics.push(lrc);
        var res = timeReg.exec(value)
        if(res == null) return true;
        var timeStr = res[1]; // 00:00.92
        var res2 = timeStr.split(":");
        var min = parseInt(res2[0]) * 60;
        var sec = parseFloat(res2[1]);
        var time = parseFloat(Number(min + sec).toFixed(2));
        _this.times.push(time);
      });
    },
    currentIndex(currentTime){
      if(currentTime >= this.times[0]){
        this.index++;
        this.times.shift();
      }
    },
    // 监听歌曲是否播放完成
    endedMusic(playerType){
        var _this = this;
        audio.addEventListener("ended", function() {
          _this.getSong(_this.music_index + 1,playerType);
        }, true);
        audio.addEventListener("timeupdate", function() {
          // 同步时间
          var duration = audio.duration;       // 总时间
          var currentTime = audio.currentTime; // 当前时间
          var timeStr = _this.formatDate(duration,currentTime);
          _this.music_timeStr = timeStr;
          // 同步进度条
          var value = currentTime / duration * 100;
          _this.setProgress(value);
          _this.currentIndex(currentTime);
        },true)
    },
    // 暂停播放
    stopMusic(){
      if(audio.paused){
        audio.play();
        this.music_play=true;
      }else{
        audio.pause();
        this.music_play=false;
      }
    },
    // 点击进度条
    progressClick(e,reset){
        // 获取背景距离窗口的位置
        var progressLeft = Index.offsetLeft + 140;                                                  // 140边框合计
        var pageX = e.pageX;
        // 不给小圆点拖出进度条外
        if(pageX < progressLeft) pageX = progressLeft;                                                // 左边
        if(pageX > (progress.offsetWidth + progressLeft)) pageX = progress.offsetWidth + progressLeft;// 右边
        progressBar.style.width = (pageX - progressLeft) + 'px';
        progressDot.style.left = (pageX - progressLeft) - (progressDot.offsetWidth / 2) + 'px'; // 4小圆点大小的一半
        // 计算进度条比例
        var progressWidth = progress.offsetWidth;                                               // 进度条背景宽度
        this.music_value = (pageX - progressLeft) / progressWidth;                              // 进度条比例
        if(reset){
          // 点击才重置时间，拖拽中不重置
          this.musicSeekTo(this.music_value);
        }
    },
    // 拖拽进度条
    progressMove(e){
      var _this = this;
      document.onmousemove=function(e){
        _this.isMove=true;
        _this.progressClick(e,false);
      }
      document.onmouseup=function(){
        document.onmousemove = null;
        document.onmouseup = null;
        _this.isMove=false;
        _this.musicSeekTo(_this.music_value);
      }
    },
    //格式化时间
    formatDate(duration,currentTime){
      var endMin = parseInt(duration / 60);
      var endSec = parseInt(duration % 60);
      if(isNaN(endMin)) endMin = 0;
      if(isNaN(endSec)) endSec = 0;
      if(endMin < 10){
        endMin = "0" + endMin;
      }
      if(endSec < 10){
        endSec = "0" + endSec;
      }

      var startMin = parseInt(currentTime / 60);
      var startSec = parseInt(currentTime % 60);
      if(startMin < 10){
        startMin = "0" + startMin;
      }
      if(startSec < 10){
        startSec = "0" + startSec;
      }
      return startMin + ":" + startSec + " / " + endMin + ":" + endSec;
    },
    // 同步进度条
    setProgress(value){
      if(this.isMove) return;
      if(value < 0 || value > 100) return;
      progressBar.style.width = value + '%';
      progressDot.style.left = value + '%';
    },
    // 设置歌曲时间
    musicSeekTo(value){
      if(isNaN(value)) return;
      audio.currentTime = audio.duration * value
    },
    voiceClick(e){
      // 获取背景距离窗口的位置
      var progressLeft = Index.offsetLeft + 884;
      var pageX = e.pageX;
      progressVoiceBar.style.width = (pageX - progressLeft) + 'px';
      // 计算进度条比例
      var progressWidth = progressVoice.offsetWidth;                                               // 进度条背景宽度
      var value = (pageX - progressLeft) / progressWidth;                              // 进度条比例
      // 点击才重置时间，拖拽中不重置
      this.musicVoiceSeekTo(value);
    },
    // 音量最大或无
    musicVoiceSet(){
      if(audio.volume > 0){
        this.music_voice = 0;
        this.musicVoiceSeekTo(0);
        progressVoiceBar.style.width = 0 + '%';
      }else{
        this.music_voice = 1;
        this.musicVoiceSeekTo(1);
        progressVoiceBar.style.width = 100 + '%';
      }
    },
    // 音量大小 0~1
    musicVoiceSeekTo(value){
      if(isNaN(value)) return;
      if(value < 0 && value > 1) return;
      this.music_voice = value;
      audio.volume = value;
    }
  }
}
</script>

<style scoped>
  a{
    text-decoration: none;
  }
  li{
    list-style: none;
  }
  i{
    font-style: normal;
  }
  img{
    display: block;
  }
  input{
    outline: none;
  }
  .player-list ol li,.more{
    cursor: pointer;
  }
  .title{
    border-bottom: 1px solid;
    padding: 30px 0;
  }
  .title p{
    padding-top: 10px;
    font-size: 12px;
  }
  .text-center{
    text-align: center;
  }
  label{
    margin-right: 15px;
  }
  .player-type span{
    padding-left: 5px;
    vertical-align: text-bottom;
  }
  .Index{
    width: 950px;
    margin: 0 auto;
  }
  .bg{
    position: fixed;
    height: 110%;
    width: 110%;
    left: -5%;
    top: -5%;
    z-index: -1;
    filter: blur(20px);
  }
  .bg,.music-pic div{
    background-repeat: no-repeat;
    background-size: cover;
    background-position: center;
  }
  .bg:after {
    content: '';
    position: absolute;
    background: rgba(0, 0, 0, .2);
    height: 100%;
    width: 100%;
  }
  .search-div,.player-type{
    padding: 30px 0;
  }
  .search{
    border: none;
    background-color: transparent;
    border: 1px solid;
  }
  .search{
    width: 100%;
    display: block;
    height: 45px;
    line-height: 45px;
    box-sizing: border-box;
  }
  .playlist{
    padding: 10px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, .3);
  }
  .player-controller{
    display: flex;
    user-select: none;
  }
  .music-pic{
    min-width: 120px;
    min-height: 120px;
    margin-right: 10px;
  }
  .music-pic div{
    width: 100%;
    height: 100%;
  }
  .music-info{
    width: calc(100% - 120px);
    height: 120px;
    position: relative;
  }
  .music-info h3{
    font-weight: 100;
  }
  .player-list{
    padding-top: 20px;
  }
  .player-list ol li{
    display: flex;
    list-style: none;
    border-top: 1px solid;
    height: 32px;
    line-height: 32px;
    font-size: 12px;
    padding: 0 12px 0 0;
    box-sizing: border-box;
    align-items: center;
  }
  .player-list ol li span{
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }
  .player-list ol li:first-child{
    border: none;
  }
  .player-list ol li span:first-child{
    width: 3px;
    height: 80%;
    background-color: transparent;
  }
  .player-list ol li span:nth-child(2){
    width: 4%;
  }
  .player-list ol li span:nth-child(3){
    width: 86%;
  }
  .player-list ol li span:last-child{
    width: 10%;
    text-align: right;
  }
  .more{
    font-size: 12px;
    padding: 20px 0;
    color: #666;
  }
  .active,.player-list ol li:hover{
    background-color: rgba(0,0,0,.05);
  }
  .player-list ol li:hover .border-left{
    display: block;
  }
  .active .border-left{
    background-color: rgba(255,255,255,.80) !important;
  }
  .title,.player-list ol li,.search{
    border-color: rgba(255,255,255,.20);
  }
  .title,.title a,.music-info h3,.player-list ol li,label,.search,.progressTime{
    color: rgba(255,255,255,.80);
  }
  .prev,.next,.pause,.play{
    cursor: pointer;
    background-position: center;
    background-size: contain;
    width: 60px;
    height: 60px;
  }
  .pause,.play{
    width: 80px;
    height: 80px;
  }
  .play{
    background-image: url(../assets/play_btn_play.png);
  }
  .pause{
    background-image: url(../assets/play_btn_pause.png);
  }
  .prev{
    background-image: url(../assets/play_btn_prev.png);
  }
  .next{
    background-image: url(../assets/play_btn_next.png);
  }
  .music_voice_progress,.progress{
    background: #cdcdcd;
    cursor: pointer;
  }
  .progress{
    position: absolute;
    height: 2px;
    width: calc(100% - 150px);
    left: 0;
    bottom: 0;
  }
  .play-btn{
    height: 80px;
    width: 200px;
    display: flex;
    align-items: center;
    bottom: 10px;
    left: 0;
  }
  .progressBar,.progressDot,.music_voice_progressBar{
    background: #C20C0C;
  }
  .progressBar{
    height: 2px;
    width: 0;
  }
  .progressDot,.progressTime,.music_voice_ground,.music_voice_progress,.play-btn{
    position: absolute;
  }
  .progressDot{
    height: 8px;
    width: 8px;
    border-radius: 100%;
    left: 0;
    top: -3px;
  }
  .progressTime{
    right: -80px;
    width: 80px;
    bottom: -5px;
    font-size: 10px;
  }
  .music_voice_icon,.bg,.music-pic div,.music_voice_progressBar{
    transition: .3s;
  }
  svg{
    fill: rgba(255,255,255,.80);
  }
  .music_voice_ground{
    width: 70px;
    right: 0;
    bottom: -5px;
  }
  .music_voice_icon{
    width: 14px;
    height: 14px;
    cursor: pointer;
  }
  .music_voice_progress{
    width: calc(100% - 14px);
    height: 2px;
    right: 0;
    bottom: 5px;
  }
  .music_voice_icon:hover{
    transform: scale(1.2);
  }
  .music_voice_progressBar{
    width: 100%;
    height: 100%;
  }
  #lyric{
    height: calc(100% - 20px);
    position: absolute;
    top: 0px;
    right: 0;
    overflow: hidden;
    line-height: 20px;
    font-size: 12px;
    text-align: center;
  }
  #lyric_ul{
    transition: all 0.5s ease-out;
    color: #cdcdcd;
  }
  .cw{
    color: #fff !important;
  }
  /* 动画 */
  .music{width: 50px;height: 50px;position: relative;margin: 0 auto;}
  .music i{width: 4px;height: 5px;position: absolute;bottom:0;background-color: rgba(255,255,255,.80);}
  .music i:nth-of-type(1){left:0;}
  .music i:nth-of-type(2){left:8px;}
  .music i:nth-of-type(3){left:16px;}
  .music i:nth-of-type(4){left:24px;}
  .music i:nth-of-type(5){left:32px;}
  .music i:nth-of-type(6){left:40px;}
  .music i:nth-of-type(1){-webkit-animation:wave 0.66s linear infinite;animation:wave 0.66s linear infinite;}
  .music i:nth-of-type(2){-webkit-animation:wave 0.8s linear infinite;animation:wave 0.8s linear infinite;}
  .music i:nth-of-type(3){-webkit-animation:wave 0.7s linear infinite;animation:wave 0.7s linear infinite;}
  .music i:nth-of-type(4){-webkit-animation:wave 0.5s linear infinite;animation:wave 0.5s linear infinite;}
  .music i:nth-of-type(5){-webkit-animation:wave 0.9s linear infinite;animation:wave 0.9s linear infinite;}
  .music i:nth-of-type(6){-webkit-animation:wave 1.2s linear infinite;animation:wave 1.2s linear infinite;}
  @-webkit-keyframes wave{
    0%{height:8px}
    50%{height: 32px}
    100%{height: 12px}
  }
  @keyframes wave{
    0%{height:8px}
    50%{height: 32px}
    100%{height: 12px}
  }
</style>