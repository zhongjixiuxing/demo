<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title style="text-align: center;">弹幕监听</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">

      <!-- <ExploreContainer name="Tab 1 page" /> -->
      <div style="
        height: 100%;
        width: 100%;
        display: flex;
        flex-direction: column;
        justify-content: center;
        margin-top: 12px;
        ">

        <div v-if="!wsClient">
          <div style="padding-left: 2vw;padding-right: 2vw;">
              <ion-input label="Web抖音直播地址" @ionInput="changeDouyinLiveUrl" label-placement="floating" fill="outline" placeholder="请输入" :value="douyinLiveUrl"></ion-input>
          </div>
          <div style="padding-left: 2vw;padding-right: 2vw;">
              <ion-input label="后端数据接受地址" @ionInput="changeRemoteApiUrl" label-placement="floating" fill="outline" placeholder="请输入" :value="remoteApiUrl"></ion-input>
          </div>
          <div style="margin-top: 12px;">
            <ion-button style="padding-left: 2vw;padding-right: 2vw;" color="primary" @click="startListen">启动监听</ion-button>
          </div>
        </div>
        <div v-if="wsClient">
          <div style="margin-top: 12px;">
            <ion-button style="padding-left: 2vw;padding-right: 2vw;" color="primary" @click="stopListen">停止监听</ion-button>
          </div>
        </div>

        <div class="douyin-msg-box" v-if="msgs.length">
          <div class="header">
            <div class="left">
              <div class="title">
                <ion-text color="secondary">
                  <h2>{{(wsClient && wsClient.anchor && wsClient.anchor.nickname) ? wsClient.anchor.nickname : ''}}</h2>
                </ion-text>
              </div>
              <div>
                <ion-text color="danger">{{(wsClient && wsClient.anchor && wsClient.anchor.id_str) ? wsClient.anchor.id_str : ''}}</ion-text> / 
                <ion-text color="danger">{{(wsClient && wsClient.roomInfo) ? wsClient.roomInfo.liveRoomId : ''}}</ion-text> / 
                <ion-text color="danger">{{msgs.length}}</ion-text>
              </div>
            </div>
            <div class="righ">{{currentTime}}</div>
          </div>
          
          <div class="messages">
            <ion-content>
              <ion-list :inset="true" style="margin: 12px 0 0 0;">
                <ion-item v-for="msg in msgs" :key="msg.common.msgId">
                   <ion-chip>
                    <ion-avatar>
                      <img alt="person's head" :src="msg.user.AvatarThumb.urlListList[0] || 'https://ionicframework.com/docs/img/demos/avatar.svg'" />
                    </ion-avatar>
                    <ion-label>{{msg.user.nickName}}</ion-label>
                  </ion-chip>
                  <ion-label>{{msg.content}}</ion-label>
                </ion-item>
              </ion-list>
            </ion-content>
          </div>
        </div>
      </div>
      
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonInput, IonChip, IonItem, IonLabel, IonList, IonAvatar, IonText, IonButton } from '@ionic/vue';
import { ref } from 'vue'
import { DouYinWS } from '../services/DouYinWS'
import { CapacitorHttp } from '@capacitor/core';

const url = 'https://live.douyin.com/794197581224'


const wsClient = ref<any>()
const douyinLiveUrl = ref<string>('')
const remoteApiUrl = ref<string>('')
let currentTime = ref<string>(new Date().toLocaleTimeString())
setInterval(() => {
  currentTime.value = new Date().toLocaleTimeString()
}, 1000)
let room = ref<any>({title: '', id: '', liveRoomId: ''})
let msgs = ref<any[]>([
//   {
//     "common": {
//         "method": "WebcastChatMessage",
//         "msgId": "7276858152351274000",
//         "roomId": "7276512287362502000",
//         "isShowMsg": true,
//         "priorityScore": "31000"
//     },
//     "user": {
//         "id": "157930975666503",
//         "shortId": "3803764089",
//         "nickName": "流心影者",
//         "gender": 1,
//         "AvatarThumb": {
//             "urlListList": [
//                 "https://p3.douyinpic.com/aweme/100x100/aweme-avatar/tos-cn-i-0813_5197abf1e5684749b97d10e3d669985e.jpeg?from=3067671334"
//             ]
//         },
//         "BadgeImageList": [
//             {
//                 "urlListList": [
//                     "http://p3-webcast.douyinpic.com/img/webcast/new_user_grade_level_v1_20.png~tplv-obj.image"
//                 ],
//                 "height": "16",
//                 "width": "32",
//                 "imageType": 1,
//                 "content": {
//                     "level": "20",
//                     "alternativeText": "荣誉等级20级勋章"
//                 }
//             }
//         ],
//         "FollowInfo": {
//             "followingCount": "239",
//             "followerCount": "7432856",
//             "followerCountStr": "0",
//             "followingCountStr": "0"
//         },
//         "displayId": "LaoSiJi_movie",
//         "secUid": "MS4wLjABAAAA5JaF7p0V8n2bqSLUEVw8dQCU0rZG9ZkStLRXKXs8z0I"
//     },
//     "content": "@毛家四哥 嗯嗯好",
//     "giftImage": {
//         "width": "80",
//         "avgColor": "\n\u00016\u0012\u00011",
//         "content": {}
//     }
// }
])

const sendToRemoteServer = async (msg: any) => {
  const options = {
    url: 'https://api.chatbot.anxing131.xyz/openai/douyin/live',
    // url: 'https://anxing.requestcatcher.com/',
    headers: {
      'Content-Type': 'application/json'
    },
    data: {
       "body": {
        "mode": "store",
        "content": msg.content
    }},
  };

  CapacitorHttp.post(options);
  if (remoteApiUrl.value) {
    const body = {
      url: remoteApiUrl.value,
      headers: {
        'Content-Type': 'application/json'
      },
      data: {
        msg: msg,
        anchor: wsClient.value.anchor
      },
    }
    await CapacitorHttp.post(body)
  }
}

async function startListen() {
  if (!douyinLiveUrl.value.trim()) {
    alert('请输入Web抖音地址')
    return
  }
  wsClient.value = new DouYinWS({
    url: douyinLiveUrl.value.trim(),
    onmessage: (incomingMsgs: any) =>{
      if (!incomingMsgs.length) {
        return 
      }
      
      for (const msg of incomingMsgs) {
        if (msg.common.method !== 'WebcastChatMessage') {
          continue
        }

        sendToRemoteServer(msg)
        msgs.value.push(msg)
      }
    },
    onroominfo: (roomInfo: any) => {
      room.value = roomInfo
    }
  })
  wsClient.value.start()
}

async function stopListen() {
  if (wsClient.value) {
    wsClient.value.wsClient.close()
    wsClient.value = null
  }
}

function changeDouyinLiveUrl(event: any) {
  douyinLiveUrl.value = event.target.value
}

function changeRemoteApiUrl(event: any) {
  remoteApiUrl.value = event.target.value
}

</script>


<style lang="scss" scoped>
#container {
  text-align: center;
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  color: #8c8c8c;
  margin: 0;
}

#container a {
  text-decoration: none;
}

.douyin-msg-box {
  display: flex;
  flex-direction: column;
  height: 100%;
  margin-left: 2vw;
  margin-right: 2vw;
  margin-top: 12px;
  padding: 4px;
  .header {
    display: flex;
    width: 100%;
    justify-content: space-between;
    align-content: space-between;
    flex-direction: row;
    .righ {
      display: flex;
      align-items: flex-end;
    }
  }

  .messages {
    height: 100%;
  }
}
</style>
