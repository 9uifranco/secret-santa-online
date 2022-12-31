<script setup>
    defineProps({
        popupActive: {
            type: Boolean,
            default: false
        }
    })

</script>

<template>
    <Teleport to="body">
        <transition name="popup">
            <div v-show="popupActive" class="fixed w-full h-full bg-black bg-opacity-30 top-0 left-0 flex justify-center font-mono border-transparent focus:border-transparent focus:ring-0">
                
                <div class="bg-sky-300 self-center h-96 min-h-[50%] text-center w-80 rounded-md flex flex-col justify-between p-5 shadow-md" >
                    
                    <slot/>
                </div>
            </div>
        </transition>
    </Teleport>
</template>

<style scoped>

.popup-enter-active, .popup-leave-active {
  transition: all 0.5s ease;
}
.popup-enter-from, .popup-leave-to {
  opacity: 0;
}

</style>