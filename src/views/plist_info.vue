<template>
  <div class="rank-info-content plist-info">
    <div class="rank-banner-wrap" :style="{'background-image':'url('+imgSrc+')','background-size': '100%','background-repeat': 'no-repeat','background-position': 'center'}">

    </div>

    <div class="plist-desp container">
      <p class="plist-desp-p" :class="{'plist-desp-hide': hideDesp }">{{desp}}</p>
      <img src="../assets/images/close_icon.png" alt="" @click="toggleDesp" class="plist-desp-icon" v-if="hideDesp">
      <img src="../assets/images/open_icon.png" alt="" @click="toggleDesp" class="plist-desp-icon" v-else>
    </div>
    <div class="plist-desp-bottom" style="width: 100%;height: 5px;background-color: #f1f1f1"></div>

    <div class="rank-info-list">
      <mt-cell v-for="(item,index) in songList" :title="item.title" @click.native="playAudio(index)">
        <img src="../assets/images/download_icon.png" alt="" width="20" height="20">
      </mt-cell>
    </div>
  </div>
</template>

<script type="es6">
  import { Indicator } from 'mint-ui'
  export default {
    data(){
      return {
        imgSrc:'',
        title:'',
        songList:[],
        updateTime:'',
        desp:'',
        hideDesp:true,
        opacity:0,
      }
    },
    //通过路由的before钩子解除router-view缓存限制
    beforeRouteEnter (to, from, next) {
      next(vm => {
        vm.$store.commit('showHead',true);
        vm.get();
        window.onscroll=()=>{
          vm.opacity=window.pageYOffset/250;
          vm.$store.commit('setHeadStyle',{background:'rgba(43,162,251,'+vm.opacity+')'})
        }
      })
    },
    beforeRouteLeave(to,from,next){
      this.$store.commit('showHead',false);
      window.onscroll=null;
      next()
    },
    methods:{
      get(){
        Indicator.open({
          text: '加载中...',
          spinnerType: 'snake'
        });
        var infoID=this.$route.params.id;
        this.$http.get('http://lavyun.applinzi.com/apis/getPage.php?path='+'/plist/list/'+infoID).then((res)=>{
          Indicator.close()
          this.parseList(res.data)
        })
      },
      parseList(data){
        var div=document.createElement('div');
        div.innerHTML=data;
        this.imgSrc=div.querySelector('#imgBoxInfo img').src;
        this.title=div.querySelector('.page-title').innerText;
        this.desp=div.querySelector('#introBox p').innerText;
        this.$store.commit('setHeadTitle',this.title);
        var list=div.querySelectorAll('.panel-songslist-item');
        this.songList = [];
        for(var i=0;i<list.length;i++){
          var song={};
          var title=list[i].querySelector('.panel-songs-item-name span').innerText;
          var hash=list[i].id.substr(6);
          song.title=title;song.hash=hash;
          this.songList.push(song)
        }
      },
      playAudio(index){
        var hash=this.songList[index].hash;
        this.$store.dispatch('getSong',hash);
        this.$store.dispatch('getLrc',hash);
      },
      toggleDesp(){
        this.hideDesp=!this.hideDesp;
      },

    }
  }
</script>

<style scoped>
  .rank-banner-wrap{height: 250px;background-size: 100%}
</style>

