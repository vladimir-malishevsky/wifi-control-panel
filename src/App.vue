<template>
  <div class="wrapper">
    <div class="controls-container">
      <Control v-for="control in controlsList" :data="control"/>
    </div>
    <div class="controls-actions">
      <a href="" @click.prevent="addControl">add</a> |
      <a href="" @click.prevent="removeControl">remove</a> 
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import Control from './components/Control.vue';

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

.controls-actions {
  text-align: center;
  color: var(--hidden-text-color);
  a {
    text-decoration: none;
    color: var(--hidden-text-color);
  }
}
</style>
