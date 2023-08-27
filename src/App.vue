<template>
  <div class="wrapper">
    <div class="controls-container">
      <div class="control" :class="[controlStatusClasses[control.status]]" v-for="control in controlsList" @click="switchControl(control)">
        <div class="control__title">{{ control.title }}</div>
      </div>
    </div>
    <div class="controls-actions">
      <a href="" @click.prevent="addControl">add</a> |
      <a href="" @click.prevent="removeControl">remove</a> 
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const controlStatusClasses = {
  undefined: 'pending',

  'pending': 'pending',
  '1': 'active',
  '0': '',
  'error': 'error'
}

const controlsList = ref([...getStoredControlsList()])

fetchControls()

function fetchControls() {
  controlsList.value.forEach(control => {
    fetch(`http://${control.ip}/status`).then(response => response.text()).then(statusCode => {
      control.status = statusCode
    }).catch(error => {
      control.status = 'error'
      console.error(error)
    })
  })
}

function switchControl(control) {
  if (control.status === 'pending') return;

  control.status = 'pending'
  fetch(`http://${control.ip}/switch`).then(response => response.text()).then(statusCode => {
    control.status = statusCode
  }).catch(error => {
    control.status = 'error'
    console.error(error)
  })
}

function addControl() {
  const newControlIp = prompt('Enter IP adress:')

  if (!newControlIp) return;

  for (const control of controlsList.value) {
    if (newControlIp == control.ip) {
      alert('This ip already exist!')
      return
    }
  }

  const newControlTitle = prompt('Enter title:', newControlIp)

  const newControl = { ip: newControlIp, title: newControlTitle }

  controlsList.value.push(newControl)
  fetchControls()
  storeControl(newControl)
}

function removeControl() {
  const targetTitle = prompt('Enter control title which you want to remove:')
  if (!targetTitle) {
    alert('Invalid title')
  }

  controlsList.value = controlsList.value.filter(control => targetTitle != control.title)

  removeStoredControl(targetTitle)
}

function getStoredControlsList() { return JSON.parse(localStorage.getItem('controlsList')) || [] }

function storeControl(newControl) {
  const storedControlsList = getStoredControlsList()
  storedControlsList.push(newControl)
  localStorage.setItem('controlsList', JSON.stringify(storedControlsList))
}

function removeStoredControl(controlTitle) {
  let storedControlsList = getStoredControlsList()
  storedControlsList = storedControlsList.filter(control => controlTitle != control.title)
  localStorage.setItem('controlsList', JSON.stringify(storedControlsList))
}
</script>

<style lang="scss" scoped>
.wrapper {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.controls-container {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.control {
  width: 400px;
  height: 50px;
  margin-bottom: 10px;

  background: #323232;

  user-select: none;

  transition: all 0.2s;

  &:hover {
    cursor: pointer;
    background-color: #424242;
  }

  &.pending {
    cursor: not-allowed;
    background: linear-gradient(to right, #323232 0%, #393939 15%, #323232 30%);

    animation: controlPending 3s linear infinite;
    
    @keyframes controlPending {
      0% {
        background-position: 0rem;
      }

      100% {
        background-position: 25rem;
      }
    }
  }
  &.active {
    background-color: #646CFE;
  }
  &.error {
    background-color: #FE6C64;
  }

  @media screen and (max-width: 400px) {
    width: 300px;
    height: 100px;
  }

  .control__title {
    width: 100%;
    height: 100%;

    display: flex;
    justify-content: center;
    align-items: center;
  }
}

.controls-actions {
  text-align: center;
  color: #323232;
  a {
    text-decoration: none;
    color: #323232;
  }
}
</style>
