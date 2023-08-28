<template>
    <div class="control" :class="[controlStatusClasses[props.data.status]]" @click="switchControl(props.data)">
        <div class="control__title">{{ props.data.title }}</div>
    </div>
</template>

<script setup>
const props = defineProps({
    data: {
        type: Object,
        required: true,
    }
})

const controlStatusClasses = {
  undefined: 'pending',

  'pending': 'pending',
  '1': 'active',
  '0': '',
  'error': 'error'
}

function switchControl(control) {
  if (control.status === 'pending') return;

  control.status = 'pending'
  fetch(`http://${control.ip}/switch`).then(response => response.text()).then(statusCode => {
    control.status = statusCode
  }).catch(error => {
    // control.status = 'error'
    setTimeout(() => {control.status = 'error'}, 5000)
    console.error(error)
  })
}
</script>

<style lang="scss" scoped>
.control {
  width: 400px;
  height: 50px;
  margin-bottom: 10px;

  background: var(--control-button-background-color);

  user-select: none;

  transition: all 0.2s;

  &:hover {
    cursor: pointer;
    background-color: #424242;
  }

  &.pending {
    cursor: not-allowed;
    background: linear-gradient(to right, var(--control-button-background-color) 0%, var(--control-button-pending-dash-color) 15%, var(--control-button-background-color) 30%);

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
</style>