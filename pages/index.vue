<template>
  <v-container>
    <v-textarea
      v-model="talktext"
      counter
      label="トーク履歴"
      no-resize
      outlined
      placeholder="LINEのトーク履歴を貼り付け"
      rows="5"
    />
    <v-btn
      elevation="2"
      @click="onClickedAnalize"
    >
      Analize
    </v-btn>
    <p>
      総メッセージ数: {{ talks.length }}
    </p>
  </v-container>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'

interface Talk {
  date: Date,
  sender: String,
  contents: String
}

@Component
export default class Home extends Vue {
  talktext: String = ''
  talks: Talk[] = []

  onClickedAnalize () {
    // リセット
    this.talks = []
    // console.log(this.talktext)
    const regexpDate = /^[0-9]{4}\/(0[1-9]|1[0-2])\/(0[1-9]|[12][0-9]|3[01])/
    const regexpTime = /^[0-9]{2}:[0-9]{2}/
    const regexpReceiver = /\[LINE\] (.*)とのトーク履歴/
    const messages = this.talktext.split(/\n/)
    let sendDate!: Date
    let date!: String
    let receiver!: String
    let sender!: String
    let contents!: String
    for (const message of messages) {
      // 送信先の名前を取得
      if (message.match(regexpReceiver)) {
        const matched = message.match(regexpReceiver)
        console.log(matched)
        receiver = String(matched![1])
        continue
      }
      // 日付の取得
      if (message.match(regexpDate)) {
        const matched = message.match(regexpDate)
        date = String(matched![0])
        // console.log(date)
        continue
      }
      // 時間から始まるメッセージ
      if (message.match(regexpTime)) {
        if (sendDate) {
          const talk: Talk = {
            date: sendDate,
            sender,
            contents
          }
          this.talks.push(talk)
        }
        const talkline = message.split(/\t/)
        if (talkline.length !== 3) {
          continue
        }
        const time = talkline[0]
        sender = talkline[1]
        contents = talkline[2]
        sendDate = new Date(date + '/' + time)
      } else {
        // 改行されたメッセージを追加
        contents = contents + message
      }
    }
    const talk: Talk = {
      date: sendDate,
      sender,
      contents
    }
    this.talks.push(talk)
    console.log(this.talks)
  }
}
</script>
