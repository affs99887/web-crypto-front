<script setup>
import { ref, onMounted } from 'vue'

const inputText = ref('')
const encryptedData = ref('')
const decryptedData = ref('')
let serverPublicKey = null

onMounted(async () => {
  await fetchServerPublicKey()
})

async function fetchServerPublicKey() {
  try {
    const response = await fetch('http://localhost:3000/public-key')
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }
    const {publicKey} = await response.json()
    console.log('Received public key (Base64):', publicKey)

    // 解码Base64字符串为PEM格式
    const pemKey = atob(publicKey)
    console.log('Decoded PEM key:', pemKey)

    // 从PEM格式提取实际的公钥数据
    const pemHeader = '-----BEGIN PUBLIC KEY-----'
    const pemFooter = '-----END PUBLIC KEY-----'
    const pemContents = pemKey.substring(
        pemKey.indexOf(pemHeader) + pemHeader.length,
        pemKey.indexOf(pemFooter)
    ).replace(/\s/g, '')

    // 将Base64编码的公钥转换为ArrayBuffer
    const binaryString = atob(pemContents)
    const bytes = new Uint8Array(binaryString.length)
    for (let i = 0; i < binaryString.length; i++) {
      bytes[i] = binaryString.charCodeAt(i)
    }

    console.log('Converted public key to ArrayBuffer:', bytes)

    serverPublicKey = await window.crypto.subtle.importKey(
        'spki',
        bytes,
        {
          name: 'RSA-OAEP',
          hash: 'SHA-256',
        },
        true,
        ['encrypt']
    )

    console.log('Successfully imported public key:', serverPublicKey)
  } catch (error) {
    console.error('获取服务器公钥失败:', error)
    console.error('错误详情:', error.message)
    if (error.name === 'DataError') {
      console.error('可能是公钥格式不正确或损坏')
    }
  }
}

async function encryptAndSend() {
  if (!serverPublicKey) {
    console.error('服务器公钥未加载')
    return
  }
  try {
    console.log('开始加密过程...');
    // 生成一个随机的AES密钥
    const aesKey = await window.crypto.subtle.generateKey(
        { name: 'AES-GCM', length: 256 },
        true,
        ['encrypt']
    )
    console.log('AES密钥生成成功');

    // 使用RSA-OAEP加密AES密钥
    const exportedAesKey = await window.crypto.subtle.exportKey('raw', aesKey)
    console.log('导出的AES密钥长度:', exportedAesKey.byteLength);
    const encryptedAesKey = await window.crypto.subtle.encrypt(
        { name: 'RSA-OAEP' },
        serverPublicKey,
        exportedAesKey
    )
    console.log('AES密钥加密成功，长度:', encryptedAesKey.byteLength);

    // 使用AES-GCM加密数据
    const iv = window.crypto.getRandomValues(new Uint8Array(12))
    console.log('生成的IV:', Array.from(iv));
    const encodedData = new TextEncoder().encode(inputText.value)
    console.log('要加密的数据长度:', encodedData.length);
    const encryptedData = await window.crypto.subtle.encrypt(
        { name: 'AES-GCM', iv: iv, tagLength: 128 },
        aesKey,
        encodedData
    )
    console.log('数据加密成功，长度:', encryptedData.byteLength);

    // 准备发送到服务器的数据
    const dataToSend = {
      encryptedAesKey: btoa(String.fromCharCode.apply(null, new Uint8Array(encryptedAesKey))),
      iv: btoa(String.fromCharCode.apply(null, iv)),
      encryptedData: btoa(String.fromCharCode.apply(null, new Uint8Array(encryptedData)))
    }
    console.log('准备发送的数据:', dataToSend);

    // 发送到服务器
    console.log('发送数据到服务器...');
    const response = await fetch('http://localhost:3000/decrypt', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(dataToSend)
    })
    const result = await response.json()
    if (result.success) {
      decryptedData.value = result.decryptedData
      console.log('服务器成功解密数据:', result.decryptedData);
    } else {
      console.error('服务器解密失败:', result.message, result.error)
    }
  } catch (error) {
    console.error('加密或通信错误:', error)
  }
}
</script>

<template>
  <div>
    <h2>试试加密</h2>
    <input v-model="inputText" placeholder="输入要加密的文本"/>
    <button @click="encryptAndSend">加密并发送</button>
    <div v-if="encryptedData">
      <h3>加密数据:</h3>
      <p>{{ encryptedData }}</p>
    </div>
    <div v-if="decryptedData">
      <h3>服务器返回的解密数据:</h3>
      <p>{{ decryptedData }}</p>
    </div>
  </div>
</template>
