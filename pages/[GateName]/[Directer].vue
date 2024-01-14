<template>
    <div class="main change" :class="{'hiden':!ishiden,'hiden2':ishiden2}" >
        <img  alt="mainchange" class="main1 " :src="`/sources/${imagename}.png`" @error="handleImageError(imagename)" />
    </div>
    <div class="main auto" :class="{'hiden':ishiden,'hiden2':ishiden2}">
        <video ref="videoPlayer" autoplay muted loop :src="`/sources/${videoname}.mp4`" @error="handleVideoError(videoname)"></video>
    </div>
    <div class="main manual" :class="{'hiden':!ishiden2,'hiden2':ishiden}">
        <video ref="videoPlayer" autoplay muted loop :src="`/sources/${videonamemanual}.mp4`" @error="handleManualError(videonamemanual)"></video>
    </div>
  </template>

<script>

export default {
  data() {
      return {
        imagename: "AHT/AHTBG",
        videoname: "AHT/AHTBG",
        videonamemanual: "AHT/AHTBG",
        ishiden: false, ishiden2: false, // f&f == auto, t&f == gatechange, f&t == manual
        location:"",
        name:"",
        count: 0,
        responseData:"",
        intervalId: null,

      };
    },
    created() {
     this.location = this.$route.params.GateName; 
     this.name = this.$route.params.Directer; 
     console.log(this.location+"  "+this.name);
     this.fetchData();
    },
    mounted() {
        this.ClearCache();
        this.intervalId = setInterval(this.incrementCount, 30000);
    },
    beforeDestroy() {
        clearInterval(this.intervalId);
    },
    methods: {
        handleImageError(imagename) {
         this.imagename = 'AHT/AHTBG'; 
         console.log(imagename);
        },
        handleVideoError(videoname){
            this.videoname = "OZ/OZ_NOEGATE_PRE_LEFT",
            console.log(videoname);
        },
        handleManualError(videonamemanual){
            this.videonamemanual = "OZ/OZ_NOEGATE_PRE_RIGHT",
            console.log(videonamemanual);
        },
        ClearCache() {
          //console.log("Pre Cleare Cache...");
          // Xóa cache trong trình duyệt
          //window.location.reload(true);
          // Xóa cache trong Service Workers (nếu có)
          if ('caches' in window) {
           caches.keys().then(function(cacheNames) {
            cacheNames.forEach(function(cacheName) {
             caches.delete(cacheName);
             console.log("Cleared Cache...");
            });
           });
          }   
        },
        incrementCount() {
          this.count++;
          console.log(this.count);
          this.fetchData();
        },
        fetchData(){
            const counter =  this.location;
            const leftright = this.name;
            const headers = new Headers();
            headers.append('Cache-Control', 'no-cache, no-store, must-revalidate');
            headers.append('Pragma', 'no-cache');
            headers.append('Expires', '0');
            const url = `https://localhost:7079/api/DigitalSignage/${counter}/${leftright}`;
            fetch(url) // Thay đổi URL thành URL của API bạn muốn gọi
            .then(response => response.json())
            .then(data  => {
                if(data.length>0){
                    const secondValue = data[0].lineCode+"/"+data[0].lineCode+"_"+(data[0].modeNow == "No"?"NOEGATE":"EGATE")+"_"+(((data[0].remark=="On time")||(data[0].remark=="Delayed")) ? "PRE":"BOARD")+"_"+leftright;
                    this.videoname= secondValue;
                    console.log(secondValue);
                }
                })
            .catch(error => {
                // Xử lý lỗi nếu có
                console.error(error);
            });
        },

    },
}
</script>

<style>
body {
    margin: 0px !important;
}
.main {
    max-height: 100vh;
    height: auto;
    overflow: hidden;
}
.hiden {
    display: none;
}
.hiden2 {
    display: none;
}

</style>