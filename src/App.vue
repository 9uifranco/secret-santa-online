<script setup>

  import { onMounted, ref, watch } from 'vue'
  import Popup from './components/Popup.vue'

  const participants = ref([])
  const participant = ref({
    name: '',
    email: ''
  })
  const alreadySomeoneSecret = ref([])

  const sentOnce = ref(false)

  const shuffleDisabled = ref(true)

  const participantNameValidated = ref(false)
  const participantEmailValidated = ref(false)

  const participantsNamesValidated = ref(true)
  const participantsEmailsValidated = ref(true)

  const nameInput = ref(null)
  const emailInput = ref(null)

  const popupActive = ref(false)
  const confirmed = ref(false)

  const sent = ref(false)

  // Validation for name input

  const validateParticipantName = (part) => {

    participantNameValidated.value = false
    part.nameValid = true
    
    // Check if is empty
    if(part.name.trim() === '') {
      part.nameValid = false
      part.nameValidMsg = 'Insert a name'
      participantNameValidated.value = false
      return
    }

    // Remove spaces from the start and end of string and make it lower case
    part.name = part.name.trim()
    part.name = part.name.toLowerCase()

    // Check if name is available
    var nameAlreadyPicked = false
    participants.value.forEach((p) => {
      if(part.name == p.name) {
        nameAlreadyPicked = true
      }    
    })
    if (nameAlreadyPicked) {
      part.nameValid = false
      part.nameValidMsg = 'Name is already being used'
      participantNameValidated.value = false
      return
    }

    participantNameValidated.value = true
    return
  }

  // Validation for email input

  const validateParticipantEmail = (part) => {

    participantEmailValidated.value = false
    part.emailValid = true

    // Check if it's empty
    if(part.email.trim() === '') {
      part.emailValid = false
      part.emailValidMsg = 'Insert an email'
      participantEmailValidated.value = false
      return
    }

    // Remove spaces from the start and end of string, make it lower case and remove all spaces 

    part.email = part.email.trim()

    part.email = part.email.toLowerCase()

    part.email = part.email.replace(/\s+/g, '')

    // Check if email is available
    var emailAlreadyPicked = false
    participants.value.forEach((p) => {
      if(part.email == p.email) {
        emailAlreadyPicked = true
      }    
    })
    if (emailAlreadyPicked) {
      part.emailValid = false
      part.emailValidMsg = 'Email is already being used'
      participantEmailValidated.value = false
      return
    }

    // Check if contains "@"
    if(!part.email.includes('@')) {
      part.emailValid = false
      part.emailValidMsg = 'Insert a valid email'
      participantEmailValidated.value = false
      return
    }

    participantEmailValidated.value = true
    return
  }

  const validateParticipant = (part) => {
    validateParticipantName(part)
    validateParticipantEmail(part)
  }

  // Watches when participants list has changed, make changes in localstorage and control availability of "shuffle & send" button

  watch(participants, (newPartList) => {

    localStorage.setItem('Participants', JSON.stringify(newPartList))

    // Enable/Disable "Send & Shuffle" button
    if(newPartList.length >= 3 && participantsNamesValidated.value && participantsEmailsValidated.value) {
      shuffleDisabled.value = false
    }
    else {
      shuffleDisabled.value = true
    }
      
  }, {deep: true})

  // Watches sentonce and save it in localstorage

  watch(sentOnce, (newVal) => {
    localStorage.setItem('Sent Once', newVal)
  })

  // Get data from local storage

  onMounted(() => {
    participants.value =  JSON.parse(localStorage.getItem('Participants')) || []
    sentOnce.value =  localStorage.getItem('Sent Once') || false
  })

  // Functions to correct name and email when focus out input from parts list

  const correctNameOnFocusOut = (part) => { 

    part.name = part.name.trim()

    part.name = part.name.toLowerCase()

    validateNames()
  }

  const correctEmailOnFocusOut = (part) => {

    part.email = part.email.trim()

    part.email = part.email.toLowerCase()

    part.email = part.email.replace(/\s+/g, '')

    validateEmails()
  }

  // Validate names from list

  const validateNames = () => {

    var isValid = true

    participants.value.forEach(part => {
      if(part.name == '') {
        part.nameValid = false
        isValid = false
      }
      else {
        part.nameValid = true
      }
    });

    participants.value.forEach(part => {
      var sameNameParts = participants.value.filter(p => p.name == part.name)
      if(sameNameParts.length > 1) {
        sameNameParts.forEach((p) => {
          p.nameValid = false
        })
        isValid = false
      }
    })

    if(!isValid) {
      participantsNamesValidated.value = false
    }
    else {
      participantsNamesValidated.value = true
    }
  }

  // Validate emails from list

  const validateEmails = () => {
    var isValid = true

    participants.value.forEach(part => {
      if((part.email == '') || !part.email.includes('@')) {
        part.emailValid = false
        isValid = false
      }
      else {
        part.emailValid = true
      }
    });

    participants.value.forEach(part => {
      var sameEmailParts = participants.value.filter(p => p.email == part.email)
      if(sameEmailParts.length > 1) {
        sameEmailParts.forEach((p) => {
          p.emailValid = false
        })
        isValid = false
      }
    })

    if(!isValid) {
      participantsEmailsValidated.value = false
    }
    else {
      participantsEmailsValidated.value = true
    }
  }

  // Push participant to list after validation

  const addParticipant = () => {
    // Validate participant
    validateParticipant(participant.value)

    if(!participantNameValidated.value || !participantEmailValidated.value) {
      return
    }

    // Focus on name input after added new participant  
    if(emailInput.value) {
      emailInput.value.blur()
      nameInput.value.focus()
    }

    // Push participant to list   
    participants.value.push({
      name: participant.value.name,
      email: participant.value.email,
      id: participants.value.length,
      nameValid: true,
      emailValid: true
    })

    // Reset inputs and set validated to false
    participant.value.name = ''
    participant.value.email = ''

    participantNameValidated.value = false
    participantEmailValidated.value = false
  }

  // Delete function

  const deleteParticipant = (part) => {
    // Remove participant from list
    participants.value = participants.value.filter(p => p !== part)
    // Reorder IDs
    participants.value.forEach(part => {
      part.id = participants.value.indexOf(part)
    })
  }

  // Popup Handlers

  const openPopup = () => {
    popupActive.value = true
  }

  const closePopup = () => {
    popupActive.value = false
    confirmed.value = false
  }

  const confirm = () => {
    confirmed.value = true
    sent.value = false
    shuffle()
  }

  // Aux function to get random numbers
  function getRandomInt(min, max) {
    min = Math.ceil(min);
    max = Math.floor(max);
    return Math.floor(Math.random() * (max - min) + min); // The maximum is exclusive and the minimum is inclusive
  }

  // Shuffle participants
  const shuffle = () => {
    alreadySomeoneSecret.value = []
    participants.value.forEach(part => {
      part.secret = ''
      part.secretName = ''
      var secretSanta = part.id
      while((secretSanta == part.id) || (alreadySomeoneSecret.value.includes(secretSanta))) { // take a number which is not the participant number and was not picked yet
        secretSanta = getRandomInt(0, participants.value.length)
        if(alreadySomeoneSecret.value.length == participants.value.length - 1) { // if is the last one
          if(!alreadySomeoneSecret.value.includes(part.id)) { // if the last one was not picked
            shuffle() // shuffle everything again
            return
          }
        }
      }
      part.secret = secretSanta
      part.secretName = participants.value[part.secret].name
      alreadySomeoneSecret.value.push(secretSanta)
    });
    sendSecrets()
    sentOnce.value = true
  }

  // Send secrets
  const sendSecrets = () => {
    const partsPromises = participants.value.map(function(part) {
      return Email.send({
        SecureToken: process.env.VUE_APP_SECURE_TOKEN,
        To: part.email,
        From: process.env.VUE_APP_EMAIL,
        Subject: "Secret Santa Online",
        Body: part.secretName
      }).then(function () {
        console.log("SENT")
      })
    });
    
    Promise.all(partsPromises).then(function () {
      setTimeout(() => {
        sent.value = true
        console.log("ALL SENT")
      }, 2000);
    })
  }

</script>

<template>
  <div class="flex flex-col center min-h-screen bg-sky-300 font-mono">

    <Popup :popupActive="popupActive">
      <transition name="fade" mode="out-in">
        <div v-if="!confirmed" class="flex flex-col justify-between h-full outline-0 focus:outline-0 focus:ring-0">
          <div class="w-full relative">
            <h1 class="font-bold text-xl self-center">Review Participants</h1>
            <img @click="closePopup" class="absolute top-0 right-0 w-3 h-3 cursor-pointer" src="./assets/delete-icon.png" alt="close-icon">
          </div>
          <div class="overflow-y-auto my-3 border rounded-md">
            <div v-for="part in participants" :key="part" class="break-words flex flex-col text-start w-full justify-between px-3 py-1 border-t first:border-t-0">
              <span class="text-sm font-bold">{{part.name}}</span>
              <span class="text-xs">{{part.email}}</span>
            </div>
          </div>
          <span v-show="sentOnce" class="p-3 text-xs text-red-500">Participants will be drawn again and a new email will be sent to each one.</span>
          <button @click="confirm" class="bg-white py-3 w-20 self-end rounded-md">Next</button>
        </div>
        <div v-else class="flex flex-col justify-between h-full outline-0 focus:outline-0 focus:ring-0">
          <transition name="fade" mode="out-in">
            <div v-if="!sent" class="flex flex-col justify-between h-full outline-0 focus:outline-0 focus:ring-0">
              <h1 class="font-bold text-xl self-center">Sending...</h1>
            </div>
            <div v-else class="flex flex-col justify-between h-full outline-0 focus:outline-0 focus:ring-0">
              <h1 class="font-bold text-xl self-center">Nice!</h1>
              <div class="text-start">
                <p class="py-2">An email has been sent to each one of the participants with their respectives Secret Santa.</p>
                <p class="pt-2 pb-4">If someone does not receive the email, these might be the problems:</p>
                <p>• Email going to SPAM folder</p>
                <p>• Incorrect email address</p>
                <p>• Mailbox full</p>
              </div>
              <button @click="closePopup" class="bg-white py-3 w-20 self-end rounded-md">Ok</button>
            </div>
          </transition>
        </div>
      </transition>
    </Popup>

    <h1 class="text-5xl font-bold text-center mb-14 mt-10">Secret Santa Online</h1>

    <div class="self-center mb-10 text-start text-xl">
      <p>- Quick and easy</p>
      <p>- Add at least 3 participants</p>
      <p>- No registration needed ;)</p>
    </div>

    <span class="self-center text-lg my-3 font-bold">Add Participant</span>
    <div class="flex justify-between self-center  border rounded-md p-3">
      <div class="flex flex-col justify-between gap-2">
        <div class="flex justify-between gap-3">
          <label>Name: </label>
          <input ref="nameInput" @keypress.enter="addParticipant"  v-model="participant.name" class="border rounded" type="text">
        </div>
        <span v-show="!participant.nameValid" class="pl-16 text-xs text-red-500">{{participant.nameValidMsg}}</span>
        <div class="flex justify-between  gap-3">
          <label>Email: </label>
          <input ref="emailInput" @keypress.enter="addParticipant"  v-model="participant.email" class="border rounded" type="email">
        </div>
        <span v-show="!participant.emailValid" class="pl-16 text-xs text-red-500">{{participant.emailValidMsg}}</span>
      </div>
      <button  @click.prevent="addParticipant" class="border-2 w-20 rounded-md ml-3 disabled:opacity-25 disabled:cursor-not-allowed bg-white hover:bg-slate-200 transition">Add</button>
    </div>

    <span class="self-center text-lg my-3 font-bold mt-10">Participants List</span>
    <transition-group class="self-center" tag="div" name="list" mode="out-in">
      <div class="flex flex-col border self-center w-80" v-for="part in participants" :key="part">
        <div class="flex justify-between">

            <span class="font-semibold p-1 text-md">Participant {{part.id + 1}}</span>

          <img @click="deleteParticipant(part)" class="w-3 h-3 m-2 cursor-pointer" src="./assets/delete-icon.png" alt="delete-icon">
        </div>
        
        <div class="flex gap-3 p-1 justify-between">
          <span class="w-16">Name: </span>
          <input @blur="correctNameOnFocusOut(part)" @input="validateNames" :class="{'border-red-500' : !part.nameValid}" class="grow border rounded" v-model="part.name"  type="text">
        </div>
        <span v-show="!part.nameValid" class="pl-20 text-xs text-red-500"> {{(part.name == '') ? 'Insert a name.' : 'Name already being used.'}}</span>
        <div class="flex gap-3 p-1 justify-between">
          <span class="w-16">Email: </span>
          <input @blur="correctEmailOnFocusOut(part)" @input="validateEmails" :class="{'border-red-500' : !part.emailValid}" class="grow border rounded" v-model="part.email" type="email">
        </div>
        <span v-show="!part.emailValid" class="pl-20 text-xs text-red-500"> {{((part.email == '') || (!part.email.includes('@'))) ? 'Insert a valid email.' : 'Email already being used.'}}</span>
      </div>
    </transition-group>

    <button :disabled="shuffleDisabled" @click.prevent="openPopup" class="border-2 self-center w-36 h-24 rounded-md my-10 disabled:opacity-25 disabled:cursor-not-allowed bg-white hover:bg-slate-200 transition">Shuffle & Send</button>

  </div> 
</template>

<style scoped>
.list-move,
.list-enter-active,
.list-leave-active{
  transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(-50px);
}

.list-leave-active {
  position: absolute;
}

.fade-enter-active, .fade-leave-active {
  transition: all 1s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

</style>
