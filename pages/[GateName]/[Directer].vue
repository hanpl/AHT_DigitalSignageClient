<template>
    <div class="main change" :class="{'hiden':!ishiden,'hiden2':ishiden2}" >
        <img  alt="mainchange" class="main1 " :src="`/sources/${imagename}.png`" @error="handleImageError(imagename)" />
    </div>
    <div class="main auto" :class="{'hiden':ishiden,'hiden2':ishiden2}">
        <video class="main1" ref="videoPlayer" autoplay muted loop :src="`/sources/${videoname}.mp4`" @error="handleVideoError(videoname)"></video>
    </div>
    <div class="main manual" :class="{'hiden':!ishiden2,'hiden2':ishiden}">
        <video class="main1" ref="videoPlayer" autoplay muted loop :src="`/sources/${videonamemanual}.mp4`" @error="handleManualError(videonamemanual)"></video>
    </div>
  </template>

<script>
import * as signalR from "@microsoft/signalr";
export default {
  data() {
      return {
        imagename: "AHT/AHTBG",
        videoname: "AHT/AHTBG",
        videonamemanual: "AHT/AHTBG",
        ishiden: false, 
        ishiden2: false, // f&f == auto, t&f == gatechange, f&t == manual
        location:"",
        name:"",
        count: 0,
        responseData:"",
        intervalId: null,
        intervalIdReload: null,
        intervalId1: null,

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
        this.intervalId = setInterval(this.incrementCount, 100000);
        this.hubConnection = new signalR.HubConnectionBuilder()
            .withUrl('http://172.17.18.12:8281/dashboardHub')
            //.withUrl('https://localhost:7248/dashboardHub')
            .configureLogging(signalR.LogLevel.Information)
            .build();
        this.hubConnection.start().then(() => {
            console.log("SignalR connection");
            this.reload();
            this.loadModeAuto();
            }).catch(err => console.error("Signalr Connection failed start:", err));
        // Sự kiện khi kết nối đến hub bị đóng lại
        this.hubConnection.onclose(() => {
            console.log("Connection closed");
            this.startInterval();
        });
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
            this.videoname = "AHT/AHTBG",
            console.log(videoname);
        },
        handleManualError(videonamemanual){
            this.videonamemanual = "AHT/AHTBG",
            console.log(videonamemanual);
        },
        ClearCache() {
          //window.location.reload(true);
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
          this.ClearCache();
          this.fetchData();
        },

        startInterval() {
            this.intervalId1 = setInterval(() => {
                this.reconnectHub();
            }, 40000);
            },
        stopInterval() {
            clearInterval(this.intervalId1);
            },
        reconnectHub() {
            this.hubConnection.start().then(() => {
            console.log("SignalR reconnection");
            this.reload();
            this.loadModeAuto();
            this.stopInterval();
            }).catch(err => console.error("Signalr ReConnection failed start:", err));
        },
        fetchData(){
            const counter =  this.location;
            const leftright = this.name;
            const headers = new Headers();
            headers.append('Cache-Control', 'no-cache, no-store, must-revalidate');
            headers.append('Pragma', 'no-cache');
            headers.append('Expires', '0');
            const url = `http://172.17.18.12:8085/api/DigitalSignage/${counter}/${leftright}`;
            //const url = `https://localhost:7079/api/DigitalSignage/${counter}/${leftright}`;
            fetch(url)
            .then(response => response.json())
            .then(data  => {
                if(data.length>0){
                    this.ishiden = data[0].gateChange != "No" ? true:false;
                    this.ishiden2 = data[0].auto != "No" ? true:false;
                    if(data[0].auto != "No"){
                        this.ishiden = false;
                    }
                    const secondValue = data[0].lineCode+"/"+data[0].lineCode+"_"+(data[0].modeNow == "No"?"NOEGATE":"EGATE")+"_"+(((data[0].remarkNo=="On time")||(data[0].remarkNo=="Delayed")) ? "PRE":"BOARD")+"_"+leftright;
                    this.videoname= secondValue;
                    this.videonamemanual = data[0].liveAuto !=""? "Manual/"+data[0].liveAuto:"AHT/AHTBG";
                    console.log(secondValue+"  "+this.videonamemanual);
                }
                else{
                    console.log("nogate");
                    this.videoname = "AHT/AHTBG";
                }
                })
            .catch(error => {
                // Xử lý lỗi nếu có
                console.error(error);
            });
        },
        reload(){
            this.hubConnection.on("ReceivedClientChanged", (connect, message) => {
            console.log(connect+message);
            this.fetchData();
            }); 
        },
        loadModeAuto(){
            this.hubConnection.on("ReceivedLoadModeAuto", (connect, message) => {
            console.log(connect+"---"+message);
            if(connect == "Yes"){
                this.ishiden = false;
                this.ishiden2 = true;
                console.log(this.ishiden+this.ishiden2);
                this.videonamemanual = message !=""? "Manual/"+message:"AHT/AHTBG";
            }
            else
            {
                this.ishiden = false;
                this.ishiden2 = false;
                this.videonamemanual = message !=""? "Manual/"+message:"AHT/AHTBG";
            }
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
.main1 {
    max-width: 100vw;
    height: auto;

}

</style>