<script lang="ts" setup>
import GameNav from "@/views/Layout/GameNav.vue";
import GameContent from '@/views/Layout/GameContent.vue'
import {getAssetsFile} from '@/utils'
import { useRouter } from 'vue-router'

const state = useGlobalState()
const router = useRouter()

function startGame() {
  router.push('/card-game')
}

function startNewGame() {
  router.push('/new-game')
}

function startMultiplayer() {
  router.push('/multiplayer')
}
</script>

<template>
  <main h-100dvh h-screen w-screen of-hidden bg-cover bg-center grid="~ rows-[max-content_1fr]"
        :style="{ backgroundImage: `url(${getAssetsFile(`${state.bgImage}.png`)})` }">
    <GameNav/>
    <div style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); z-index: 100; display: flex; flex-direction: column; gap: 20px;">
      <button 
        @click="startGame"
        style="padding: 20px 40px; font-size: 24px; background: #4caf50; color: white; border: none; border-radius: 10px; cursor: pointer; font-weight: bold; box-shadow: 0 4px 15px rgba(0,0,0,0.3);"
      >
        旧版卡牌游戏
      </button>
      <button 
        @click="startNewGame"
        style="padding: 20px 40px; font-size: 24px; background: #ff9800; color: white; border: none; border-radius: 10px; cursor: pointer; font-weight: bold; box-shadow: 0 4px 15px rgba(0,0,0,0.3);"
      >
        新版卡牌游戏（15张卡）
      </button>
      <button 
        @click="startMultiplayer"
        style="padding: 20px 40px; font-size: 24px; background: #9c27b0; color: white; border: none; border-radius: 10px; cursor: pointer; font-weight: bold; box-shadow: 0 4px 15px rgba(0,0,0,0.3);"
      >
        🌐 联机对战
      </button>
    </div>
    <GameContent/>
  </main>
</template>