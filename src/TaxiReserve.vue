<template>
  <div id="taxiReserveWindow">
    <h2 class="text-center mt-5 mb-4">タクシー配車予約</h2>
    <form class="row row-cols-1 g-3" novalidate>
      <div class="col input-group">
        <span id="basic-addon1" class="input-group-text">名前</span>
        <input v-model="taxiUserName" type="text" class="form-control" />
      </div>
      <div class="col input-group">
        <span id="basic-addon1" class="input-group-text">電話番号</span>
        <input
          v-model="taxiUserPhoneNumber"
          type="text"
          class="form-control"
          :class="{ 'is-invalid': v$.taxiUserPhoneNumber.$error }"
        />
        <div class="invalid-feedback">電話番号が入力されていません</div>
      </div>
      <div class="col">
        <label class="form-label" for="departurePlace">乗車場所</label>
        <select
          v-model="selectedDeparturePlace"
          class="form-select"
          name="departurePlace"
          @change="getTicketNumber"
          :class="{ 'is-invalid': v$.selectedDeparturePlace.$error }"
        >
          <option v-for="place in places" :key="place.id" :value="place.id">
            {{ place.name }}
          </option>
        </select>
        <div class="invalid-feedback">選択してください</div>
      </div>
      <div class="col">
        <label class="form-label" for="arrivalPlace">降車場所</label>
        <select
          v-model="selectedArrivalPlace"
          class="form-select"
          name="arrivalPlace"
          @change="getTicketNumber"
          :class="{ 'is-invalid': v$.selectedArrivalPlace.$error }"
        >
          <option v-for="place in places" :key="place.id" :value="place.id">
            {{ place.name }}
          </option>
        </select>
        <div class="invalid-feedback">選択してください</div>
      </div>
      <div class="col" v-if="isTicketMessageWindow">
        <div class="alert alert-info">
          必要なチケット枚数は {{ selectedTicketNumber }} 枚です
        </div>
      </div>
      <div class="col">
        <label for="taxiDeparturePlace" class="form-label">乗車人数</label>
        <select
          v-model="taxiNumberOfPassenger"
          class="form-select"
          :class="{ 'is-invalid': v$.taxiNumberOfPassenger.$error }"
        >
          <option value="1" selected>1</option>
          <option value="2">2</option>
          <option value="3">3</option>
          <option value="4">4</option>
        </select>
        <div class="invalid-feedback">選択してください</div>
      </div>
      <div class="col input-group">
        <span id="basic-addon1" class="input-group-text">同乗者</span>
        <input
          v-model="taxiPassengers"
          type="text"
          class="form-control"
          placeholder=""
          aria-label="Passengers"
          aria-describedby="basic-addon1"
        />
      </div>
      <div class="col input-group">
        <button
          type="button"
          class="w-100 btn btn-primary btn-lg"
          @click="reserve"
        >
          予約
        </button>
      </div>
    </form>
    <footer class="pt-5 text-muted text-center text-small">
      <p class="mb-1">&copy; 2021 温泉MaaS</p>
      <ul class="list-inline">
        <li class="list-inline-item"><a href="#">プライバシーポリシー</a></li>
        <li class="list-inline-item"><a href="#">規約</a></li>
        <li class="list-inline-item"><a href="#">サポート</a></li>
      </ul>
    </footer>
  </div>
</template>

<script>
import liff from '@line/liff'
import axios from 'axios'
import useVuelidate from '@vuelidate/core'
import { required } from '@vuelidate/validators'

export default {
  data() {
    return {
      displayName: '',
      userId: '',
      taxiUserName: '',
      taxiUserPhoneNumber: '',
      taxiNumberOfPassenger: '',
      taxiPassengers: '',
      isMessageWindow: false,
      isTicketMessageWindow: false,
      textMessageWindow: '',
      selectedDeparturePlace: '',
      selectedArrivalPlace: '',
      selectedTicketNumber: '',
      ticket: [
        { number: 1 },
        { number: 2 },
        { number: 3 },
        { number: 4 },
        { number: 5 },
        { number: 6 },
      ], // 1->2, 1->3, 1->4, 2->3, 2->4, 3->4
      places: [
        { id: '1', name: '観光会館' },
        { id: '2', name: '○○駅' },
        { id: '3', name: '○○温泉' },
        { id: '4', name: '○○カフェ' },
      ],
    }
  },

  // ページを開いた時に実行
  mounted: async function () {
      console.log('liffId:')
      console.log(process.env.VUE_APP_LIFFID)
    try {
      await liff.init({ liffId: process.env.VUE_APP_LIFFID })
      if (liff.isLoggedIn()) {
        await this.getProfile()
      } else {
        liff.login()
      }
    } catch (e) {
      console.log('エラー：Liff IDを取得できません。' + e)
    }
  },
  methods: {
    // プロフィール取得関数
    async getProfile() {
      try {
        const profile = await liff.getProfile()
        this.taxiUserName = profile.displayName // LINEの名前
        this.userId = profile.userId // LINEのID
      } catch (e) {
        console.log('関数:getProfile エラー:[' + e + ']')
      }
    },

    // ログアウト処理の関数
    logout() {
      if (liff.isLoggedIn()) {
        alert('ログアウトします。')
        liff.logout()
        window.location.reload()
      }
    },

    // 予約の関数
    async reserve() {
      // バリデーション実行
      const isFormCorrect = await this.v$.$validate()
      if (!isFormCorrect) {
        console.log('バリデーションエラー')
        return
      }

      const taxiReservation = {
        userIdToken: liff.getIDToken(),
        userName: this.taxiUserName,
        departurePlace: this.selectedDeparturePlace,
        arrivalPlace: this.selectedArrivalPlace,
        userPhoneNumber: this.taxiUserPhoneNumber,
        userNumberOfPassenger: Number(this.taxiNumberOfPassenger),
        userPassengers: this.taxiPassengers,
        numberOfTickets: Number(this.selectedTicketNumber),
        reservationDatetime: new Date().toISOString(),
      }

      // taxireserve の API を実行
      try {
        const response = await axios.post(
          '/api/taxireserve',
          JSON.stringify(taxiReservation),
        )
        await this.sendMessage({
          userId: response.data.userId,
          messageText: 'タクシー配車予約を受け付けました。',
        })
        liff.closeWindow()
      } catch (e) {
        console.log(e)
      }
    },

    // LINEにメッセージを送信する関数
    async sendMessage(messageParams) {
      if (!liff.isLoggedIn()) {
        return
      }
      const message = {
        userId: messageParams.userId,
        messageText: messageParams.messageText,
      }

      // sendmessage の API を実行
      try {
        await axios.post('/api/sendmessage', JSON.stringify(message))
      } catch (e) {
        console.log(e)
      }
    },

    getTicketNumber() {
      let idx1 = Number(this.selectedDeparturePlace)
      let idx2 = Number(this.selectedArrivalPlace)
      if (idx1 === 0 || idx2 === 0) {
        this.selectedTicketNumber = 0
        return
      }
      if (idx1 > 4 || idx2 > 4) {
        this.selectedTicketNumber = 0
        return
      }
      if (idx1 === idx2) {
        this.selectedTicketNumber = 0
        return
      }
      if (idx1 > idx2) {
        let tmpIdx = idx2
        idx2 = idx1
        idx1 = tmpIdx
      }
      this.selectedTicketNumber =
        this.ticket[(idx1 - 1) * 4 - (idx1 * (idx1 + 1)) / 2 + idx2 - 1].number
      this.isTicketMessageWindow = true
    },
  },
  setup: () => ({ v$: useVuelidate() }),
  validations() {
    return {
      taxiUserPhoneNumber: { required },
      selectedDeparturePlace: { required },
      selectedArrivalPlace: { required },
      taxiNumberOfPassenger: { required },
    }
  },
}
</script>

<style scoped>
.input-group-text {
  min-width: 6em;
}

button.w-100 {
  letter-spacing: 2em;
  text-indent: 2em;
}
</style>
