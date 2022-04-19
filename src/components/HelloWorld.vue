<script setup>
import { reactive } from 'vue'
import Web3 from 'web3'

// 钱包对象
var web3js = null
// 账户地址
var walletAddr = null
// 合约对象
var contract = null

const data = reactive({
  network: 'http://ethtest.atreez.com:8545',
  adderss: '0xf128468a5F62FADE805bddE193d81998d82e1949',
  abi: "[{\"anonymous\":false,\"inputs\":[{\"indexed\":true,\"internalType\":\"address\",\"name\":\"previousOwner\",\"type\":\"address\"},{\"indexed\":true,\"internalType\":\"address\",\"name\":\"newOwner\",\"type\":\"address\"}],\"name\":\"OwnershipTransferred\",\"type\":\"event\"},{\"inputs\":[],\"name\":\"renounceOwnership\",\"outputs\":[],\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"inputs\":[{\"internalType\":\"address\",\"name\":\"newOwner\",\"type\":\"address\"}],\"name\":\"transferOwnership\",\"outputs\":[],\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"inputs\":[{\"internalType\":\"address\",\"name\":\"newAddress\",\"type\":\"address\"}],\"name\":\"updateCurrentMedalAddr\",\"outputs\":[],\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"inputs\":[{\"internalType\":\"address\",\"name\":\"newAddress\",\"type\":\"address\"}],\"name\":\"updateHabit2UsersAddr\",\"outputs\":[],\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"inputs\":[{\"internalType\":\"address\",\"name\":\"newAddress\",\"type\":\"address\"}],\"name\":\"updateHabitAddr\",\"outputs\":[],\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"inputs\":[{\"internalType\":\"address\",\"name\":\"newAddress\",\"type\":\"address\"}],\"name\":\"updateHabitServiceAddr\",\"outputs\":[],\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"inputs\":[{\"internalType\":\"address\",\"name\":\"newAddress\",\"type\":\"address\"}],\"name\":\"updateMedalServiceAddr\",\"outputs\":[],\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"inputs\":[{\"internalType\":\"address\",\"name\":\"newAddress\",\"type\":\"address\"}],\"name\":\"updateObjectsAddr\",\"outputs\":[],\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"inputs\":[{\"internalType\":\"address\",\"name\":\"newAddress\",\"type\":\"address\"}],\"name\":\"updateUser2HabitsAddr\",\"outputs\":[],\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"inputs\":[{\"internalType\":\"address\",\"name\":\"newAddress\",\"type\":\"address\"}],\"name\":\"updateUser2MedalsAddr\",\"outputs\":[],\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"inputs\":[{\"internalType\":\"address\",\"name\":\"newAddress\",\"type\":\"address\"}],\"name\":\"updateUserAddr\",\"outputs\":[],\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"inputs\":[{\"internalType\":\"address\",\"name\":\"newAddress\",\"type\":\"address\"}],\"name\":\"updateUserOwnHabitsAddr\",\"outputs\":[],\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"inputs\":[{\"internalType\":\"address\",\"name\":\"newAddress\",\"type\":\"address\"}],\"name\":\"updateUserServiceAddr\",\"outputs\":[],\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"inputs\":[],\"name\":\"getCurrentHabit2UsersAddr\",\"outputs\":[{\"internalType\":\"address\",\"name\":\"\",\"type\":\"address\"}],\"stateMutability\":\"view\",\"type\":\"function\"},{\"inputs\":[],\"name\":\"getCurrentHabitAddr\",\"outputs\":[{\"internalType\":\"address\",\"name\":\"\",\"type\":\"address\"}],\"stateMutability\":\"view\",\"type\":\"function\"},{\"inputs\":[],\"name\":\"getCurrentHabitServiceAddr\",\"outputs\":[{\"internalType\":\"address\",\"name\":\"\",\"type\":\"address\"}],\"stateMutability\":\"view\",\"type\":\"function\"},{\"inputs\":[],\"name\":\"getCurrentMedalAddr\",\"outputs\":[{\"internalType\":\"address\",\"name\":\"\",\"type\":\"address\"}],\"stateMutability\":\"view\",\"type\":\"function\"},{\"inputs\":[],\"name\":\"getCurrentMedalServiceAddr\",\"outputs\":[{\"internalType\":\"address\",\"name\":\"\",\"type\":\"address\"}],\"stateMutability\":\"view\",\"type\":\"function\"},{\"inputs\":[],\"name\":\"getCurrentObjectsAddr\",\"outputs\":[{\"internalType\":\"address\",\"name\":\"\",\"type\":\"address\"}],\"stateMutability\":\"view\",\"type\":\"function\"},{\"inputs\":[],\"name\":\"getCurrentUser2HabitsAddr\",\"outputs\":[{\"internalType\":\"address\",\"name\":\"\",\"type\":\"address\"}],\"stateMutability\":\"view\",\"type\":\"function\"},{\"inputs\":[],\"name\":\"getCurrentUser2MedalsAddr\",\"outputs\":[{\"internalType\":\"address\",\"name\":\"\",\"type\":\"address\"}],\"stateMutability\":\"view\",\"type\":\"function\"},{\"inputs\":[],\"name\":\"getCurrentUserAddr\",\"outputs\":[{\"internalType\":\"address\",\"name\":\"\",\"type\":\"address\"}],\"stateMutability\":\"view\",\"type\":\"function\"},{\"inputs\":[],\"name\":\"getCurrentUserOwnHabitsAddr\",\"outputs\":[{\"internalType\":\"address\",\"name\":\"\",\"type\":\"address\"}],\"stateMutability\":\"view\",\"type\":\"function\"},{\"inputs\":[],\"name\":\"getCurrentUserServiceAddr\",\"outputs\":[{\"internalType\":\"address\",\"name\":\"\",\"type\":\"address\"}],\"stateMutability\":\"view\",\"type\":\"function\"},{\"inputs\":[],\"name\":\"owner\",\"outputs\":[{\"internalType\":\"address\",\"name\":\"\",\"type\":\"address\"}],\"stateMutability\":\"view\",\"type\":\"function\"}]",
  funcs: [],
})
const init = () => {
  var abiJson = JSON.parse(data.abi)
  // abiJson = abiJson.filter(element => element.anonymous !== false && element.type === 'function')
  abiJson = abiJson.filter(element => element.type === 'function')
  if (typeof window.ethereum === "undefined") {
    //没安装MetaMask钱包进行弹框提示
    const error = "Looks like you need a Dapp browser to get started.\nConsider installing MetaMask!"
    alert(error)
    console.log(error)
  } else {
    //如果用户安装了MetaMask，你可以要求他们授权应用登录并获取其账号
    // eslint-disable-next-line no-undef
    ethereum
      .enable()
      .then(function (accounts) {
        //如果用户同意了登录请求，你就可以拿到用户的账号
        walletAddr = accounts[0];
        console.log(walletAddr)
        web3js = new Web3(window.ethereum)
        contract = new web3js.eth.Contract(
          abiJson,
          data.adderss
        )
        console.log(contract)
        data.funcs = abiJson
      })
      .catch(function (reason) {
        data.funcs = []
        alert(reason)
        console.log(reason)
      });
  }
}
// eslint-disable-next-line no-unused-vars
const call = (item) => {
  console.log(item)
  call1(item)
  .then(response => {
    alert("result: " + response)
    console.log("result: ", response)  
    console.log("\n\n")
  })
  .catch(error => {
    alert("error: " + error)
    console.log("error: ", error)
    console.log("\n\n")
  })
}
const call1 = (item) => {
  const name = item.name
  console.log('name: ', name)
  const inputs = item.inputs.map(element => element.value)
  console.log('inputs: ', inputs)
  const from = walletAddr
  console.log('from: ', from)

  const fn = contract.methods[name]
  if (item.stateMutability === 'view' || item.stateMutability === 'pure') {
    console.log('method: call')
    return fn(...inputs).call({ from: from })
  } else {
    console.log('method: send')
    return fn(...inputs).send({ from: from })
  }
}
</script>

<template>
  <div style="
      margin: auto;
      width: 400px;
      height: 100%;
      display: flex;
      flex-direction: column;
      align-items: flex-start;
    ">
    <p>Network</p>
    <input v-model="data.network" />
    <p>Address</p>
    <input v-model="data.adderss" />
    <p>ABI</p>
    <input v-model="data.abi" />
    <button style="margin-top: 30px; height: 25px; width: 147px;" @click="init">Init</button>
    <div v-if="data.funcs.length > 0"
      style="background-color: lightgray; width: 100%; height: 1px; margin: 30px 0 0;" />
    <div v-for="item in data.funcs" :key="item.name" style="
    width: 100%;
    display: flex;
    flex-direction: column;
    align-items: flex-start;">
      <p style="font-weight: bold; font-size: 20px;">{{ `${item.name} (${item.stateMutability})` }}</p>
      <div v-for="input in item.inputs" :key="input.name" style="
      width: 100%;
      display: flex;
      flex-direction: column;
      align-items: flex-start;">
        <span>{{ `${input.name} (${input.type})` }}</span>
        <input v-model="input.value" />
      </div>
      <button style="margin: 10px 0 20px; height: 25px; width: 147px;" @click="call(item)">Call</button>
    </div>
  </div>
</template>

<style>
p {
  margin: 20px 0 5px;
}
</style>
