<script setup lang="ts">
import { computed, ref, watch } from 'vue'
import PKCE from 'js-pkce'
import Cookies from 'js-cookie'
import axios from 'axios'

defineProps<{ msg: string }>()

const computedCookie = (key: string) => computed({
  get: () => Cookies.get(key),
  set: (value) => {
    if (value) {
      Cookies.set(key, value)
    } else {
      Cookies.remove(key)
    }
  }
})

const accessTokenCookie = computedCookie('accessToken')
const accessToken = ref(accessTokenCookie.value)
watch(accessToken, () => {
  accessTokenCookie.value = accessToken.value
})

const user = ref()

const clientId = import.meta.env.VITE_CLIENT_ID
const appUri = import.meta.env.VITE_APP_URI
const apiUri = import.meta.env.VITE_API_URI

const pkce = new PKCE({
  client_id: clientId,
  redirect_uri: `${appUri}/oauth/callback`,
  authorization_endpoint: `${apiUri}/oauth/authorize`,
  token_endpoint: `${apiUri}/oauth/token`,
  requested_scopes: '*',
})

const url = new URL(self.location.href)
const code = url.searchParams.get('code')

const authorize = () => {
  const url = pkce.authorizeUrl()
  self.location.assign(url)
}
const fetchToken = async () => {
  const url = self.location.href
  const token = await pkce.exchangeForAccessToken(url)
  accessToken.value = token.access_token
}
const fetchUser = async () => {
  const response = await axios.get('http://localhost/api/user', {
    headers: {
      Authorization: `Bearer ${accessToken.value}`
    }
  })
  user.value = response.data
}
</script>

<template>
  <h1>{{ msg }}</h1>

  <p>
    アクセストークン
  </p>
  <p style="border: dashed gray; background: lightgray; overflow-wrap: break-word;">
    <span v-if="accessToken">
      {{ accessToken }}
    </span>
    <span v-else>
      アクセストークンが取得されていません
    </span>
  </p>

  <p>
    Step1:
    <button type="button" @click="authorize">ユーザ認証</button>
  </p>
  <p>
    Step2:
    <button :disabled="!code" type="button" @click="fetchToken">アクセストークン取得</button>
  </p>

  <p>
    Step3:
    <button :disabled="!accessToken" type="button" @click="fetchUser">ユーザ情報取得</button>
  </p>
  <p style="background: lightgray; overflow-wrap: break-word;">
      {{ user }}
  </p>
</template>

<style scoped>
a {
  color: #42b983;
}

label {
  margin: 0 0.5em;
  font-weight: bold;
}

code {
  background-color: #eee;
  padding: 2px 4px;
  border-radius: 4px;
  color: #304455;
}
</style>
