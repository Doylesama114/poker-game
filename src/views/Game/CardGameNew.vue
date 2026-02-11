<script setup lang="ts">
import { useGame } from '@/composables/useGameNew'
import type { ReforgeOption, Card } from '@/types/game'

const { 
  gameState, 
  currentPlayer, 
  opponent, 
  aiHiddenCards, 
  reforgeState, 
  hasPlayedThisTurn, 
  canPlayExtra,
  initGame, 
  choosePlay, 
  chooseReforge, 
  selectCardToPlay,
  selectSlotToPlay,
  selectTacticTarget,
  selectReforgeCard, 
  executeReforge, 
  endTurn 
} = useGame()

const reforgeOptions = ref<ReforgeOption[]>([])

function selectReforgeOption(option: ReforgeOption) {
  if (reforgeOptions.value.length < 2) {
    reforgeOptions.value.push(option)
    
    if (reforgeOptions.value.length === 2) {
      if (reforgeOptions.value.includes('redraw') && reforgeState.value.selectedCard === null) {
        return
      }
      
      executeReforge([reforgeOptions.value[0], reforgeOptions.value[1]])
      reforgeOptions.value = []
    }
  }
}

function onHandCardClick(index: number) {
  if (reforgeState.value.active && reforgeOptions.value.includes('redraw') && reforgeState.value.selectedCard === null) {
    selectReforgeCard(index)
    
    if (reforgeOptions.value.length === 2) {
      executeReforge([reforgeOptions.value[0], reforgeOptions.value[1]])
      reforgeOptions.value = []
    }
  } else if (gameState.value.phase === 'action' && currentPlayer.value.id === 'player' && !reforgeState.value.active) {
    selectCardToPlay(index)
  }
}

function getTotalPower(playerIndex: number) {
  const player = gameState.value.players[playerIndex]
  let totalPower = player.bonusPower
  player.field.forEach(slot => {
    if (slot.card && !slot.isExtra) {
      totalPower += slot.card.currentPower
    }
  })
  return totalPower
}

function getPowerColor(card: Card): string {
  if (card.currentPower > card.basePower) return 'green'
  if (card.currentPower < card.basePower) return 'red'
  return 'white'
}

function getCardTypeDisplay(card: Card): string {
  if (card.type === 'environment') return '环境'
  if (card.type === 'tactic') return '战术'
  return ''
}

function isSlotAvailable(slotIndex: number): boolean {
  return gameState.value.availableSlots?.includes(slotIndex) || false
}

function isCardPlayable(index: number): boolean {
  if (gameState.value.phase !== 'action' || currentPlayer.value.id !== 'player' || reforgeState.value.active) {
    return false
  }
  
  const card = gameState.value.players[0].hand[index]
  if (!card) return false
  
  if (hasPlayedThisTurn.value && !canPlayExtra.value) {
    return false
  }
  
  return currentPlayer.value.currentCost >= card.cost
}
</script>

<template>
  <div class="game-container">
    <!-- 游戏信息栏 -->
    <div class="game-info">
      <div class="round-info">
        <span>回合: {{ gameState.round }}</span>
        <span v-if="gameState.isFinalRound" class="final-round">最后一回合！</span>
      </div>
      <div class="message">{{ gameState.message }}</div>
    </div>

    <!-- AI区域 -->
    <div class="player-area ai-area">
      <div class="player-header">
        <h3>{{ gameState.players[1].name }}</h3>
        <div class="stats">
          <span>费用: {{ gameState.players[1].currentCost }}</span>
          <span class="power-display">总战力: <strong>{{ getTotalPower(1) }}</strong></span>
          <span>手牌: {{ gameState.players[1].hand.length }}</span>
          <span>牌组: {{ gameState.players[1].deck.length }}</span>
        </div>
      </div>
      
      <!-- AI场上 -->
      <div class="field">
        <div class="field-label">场上</div>
        <div class="field-grid">
          <div 
            v-for="(slot, index) in gameState.players[1].field.filter(s => !s.isExtra)" 
            :key="index" 
            class="field-slot"
            :class="{ 'has-card': slot.card }"
          >
            <div v-if="slot.card" class="field-card">
              <div class="card-name-small">{{ slot.card.name }}</div>
              <div class="card-power" :style="{ color: getPowerColor(slot.card) }">
                {{ slot.card.currentPower }}
              </div>
            </div>
            <div v-else class="empty-slot">空</div>
            
            <!-- 额外槽位 -->
            <div v-if="gameState.players[1].field.find(s => s.isExtra && s.parentSlot === index)" class="extra-slot-container">
              <div 
                v-for="extraSlot in gameState.players[1].field.filter(s => s.isExtra && s.parentSlot === index)"
                :key="extraSlot.position"
                class="extra-slot"
              >
                <div v-if="extraSlot.card" class="field-card extra">
                  <div class="card-name-small">{{ extraSlot.card.name }}</div>
                  <div class="card-power" :style="{ color: getPowerColor(extraSlot.card) }">
                    {{ extraSlot.card.currentPower }}
                  </div>
                </div>
                <div v-else class="empty-slot extra">额外</div>
              </div>
            </div>
          </div>
          
          <!-- 隐藏卡牌 -->
          <div v-for="(item, index) in aiHiddenCards" :key="'hidden-' + index" class="field-slot has-card">
            <div class="field-card hidden">
              <div class="card-back">?</div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 玩家区域 -->
    <div class="player-area player-area-main">
      <div class="player-header">
        <h3>{{ gameState.players[0].name }}</h3>
        <div class="stats">
          <span>费用: {{ gameState.players[0].currentCost }}</span>
          <span class="power-display">总战力: <strong>{{ getTotalPower(0) }}</strong></span>
          <span>手牌: {{ gameState.players[0].hand.length }}</span>
          <span>牌组: {{ gameState.players[0].deck.length }}</span>
        </div>
      </div>

      <!-- 玩家场上 -->
      <div class="field">
        <div class="field-label">场上</div>
        <div class="field-grid">
          <div 
            v-for="(slot, index) in gameState.players[0].field.filter(s => !s.isExtra)" 
            :key="index" 
            class="field-slot"
            :class="{ 
              'has-card': slot.card,
              'selectable': isSlotAvailable(index),
              'selected': gameState.selectedSlot === index
            }"
            @click="gameState.phase === 'selectSlot' && isSlotAvailable(index) && selectSlotToPlay(index)"
          >
            <div v-if="slot.card" class="field-card">
              <div class="card-name-small">{{ slot.card.name }}</div>
              <div class="card-power" :style="{ color: getPowerColor(slot.card) }">
                {{ slot.card.currentPower }}
              </div>
            </div>
            <div v-else class="empty-slot">{{ index + 1 }}</div>
            
            <!-- 额外槽位 -->
            <div v-if="gameState.players[0].field.find(s => s.isExtra && s.parentSlot === index)" class="extra-slot-container">
              <div 
                v-for="extraSlot in gameState.players[0].field.filter(s => s.isExtra && s.parentSlot === index)"
                :key="extraSlot.position"
                class="extra-slot"
                :class="{
                  'selectable': isSlotAvailable(extraSlot.position),
                  'selected': gameState.selectedSlot === extraSlot.position
                }"
                @click="gameState.phase === 'selectSlot' && isSlotAvailable(extraSlot.position) && selectSlotToPlay(extraSlot.position)"
              >
                <div v-if="extraSlot.card" class="field-card extra">
                  <div class="card-name-small">{{ extraSlot.card.name }}</div>
                  <div class="card-power" :style="{ color: getPowerColor(extraSlot.card) }">
                    {{ extraSlot.card.currentPower }}
                  </div>
                </div>
                <div v-else class="empty-slot extra">额外</div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 玩家手牌 -->
      <div class="hand">
        <div class="hand-label">
          手牌
          <span v-if="reforgeState.active && reforgeOptions.includes('redraw') && reforgeState.selectedCard === null" class="hint">
            (点击选择要放回牌组的卡牌)
          </span>
          <span v-else-if="!reforgeState.active && hasPlayedThisTurn && !canPlayExtra" class="hint-disabled">
            (本回合已出牌)
          </span>
          <span v-else-if="!reforgeState.active && canPlayExtra" class="hint-extra">
            (可以额外出一张牌！)
          </span>
        </div>
        <div class="hand-cards">
          <div 
            v-for="(card, index) in gameState.players[0].hand" 
            :key="index" 
            class="hand-card"
            :class="{ 
              'playable': isCardPlayable(index),
              'disabled': !isCardPlayable(index) && !reforgeState.active,
              'selectable': reforgeState.active && reforgeOptions.includes('redraw') && reforgeState.selectedCard === null,
              'selected': reforgeState.selectedCard === index
            }"
            @click="onHandCardClick(index)"
          >
            <div class="card-header">
              <span class="card-attribute">{{ card.attribute }}</span>
              <span class="card-cost-power">
                <span v-if="card.type === 'environment'" class="card-type-badge">环境</span>
                <span v-else-if="card.type === 'tactic'" class="card-type-badge">战术</span>
                <span>⚡{{ card.cost }}</span>
                <span v-if="card.type === 'unit'">💪{{ card.basePower }}</span>
              </span>
            </div>
            <div class="card-name">{{ card.name }}</div>
            <div class="card-keywords">{{ card.keywords.join('/') }}</div>
            <div class="card-effect">{{ card.effects[0]?.description || '无效果' }}</div>
          </div>
        </div>
      </div>

      <!-- 目标选择 -->
      <div v-if="gameState.phase === 'selectTarget' && gameState.availableTargets" class="target-selection">
        <div class="target-label">选择目标：</div>
        <div class="target-cards">
          <div 
            v-for="(target, index) in gameState.availableTargets"
            :key="index"
            class="target-card"
            @click="selectTacticTarget(target)"
          >
            {{ target.name }} (战力: {{ target.currentPower }})
          </div>
        </div>
      </div>

      <!-- 操作按钮 -->
      <div class="actions">
        <button 
          v-if="gameState.phase === 'draw' && gameState.round === 0"
          @click="initGame"
          class="btn btn-primary"
        >
          开始游戏
        </button>

        <template v-if="gameState.phase === 'decision' && currentPlayer.id === 'player'">
          <button @click="choosePlay" class="btn btn-primary">出牌</button>
          <button @click="chooseReforge" class="btn btn-secondary">重铸</button>
        </template>

        <template v-if="gameState.phase === 'action' && currentPlayer.id === 'player' && !reforgeState.active">
          <button @click="endTurn" class="btn btn-secondary">结束回合</button>
        </template>

        <!-- 重铸选项 -->
        <div v-if="reforgeState.active && reforgeOptions.length < 2" class="reforge-options">
          <div class="reforge-info">
            选择操作 ({{ reforgeOptions.length }}/2)
            <span v-if="reforgeOptions.includes('redraw') && reforgeState.selectedCard === null" class="warning">
              - 请先选择要换掉的手牌
            </span>
          </div>
          <button @click="selectReforgeOption('gainCost')" class="btn btn-small">恢复2费用</button>
          <button @click="selectReforgeOption('redraw')" class="btn btn-small">换牌</button>
          <button @click="selectReforgeOption('gainPower')" class="btn btn-small">战力+1</button>
        </div>

        <button 
          v-if="gameState.phase === 'gameOver'"
          @click="initGame"
          class="btn btn-primary"
        >
          重新开始
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.game-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
  color: white;
  max-width: 1600px;
  margin: 0 auto;
}

.game-info {
  background: rgba(0, 0, 0, 0.3);
  padding: 15px;
  border-radius: 10px;
  margin-bottom: 20px;
}

.round-info {
  display: flex;
  gap: 20px;
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 10px;
}

.final-round {
  color: #ff6b6b;
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.message {
  font-size: 16px;
  line-height: 1.5;
  white-space: pre-line;
}

.player-area {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 15px;
  padding: 20px;
  margin-bottom: 20px;
}

.ai-area {
  border: 2px solid rgba(255, 100, 100, 0.5);
}

.player-area-main {
  border: 2px solid rgba(100, 255, 100, 0.5);
}

.player-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.player-header h3 {
  margin: 0;
  font-size: 24px;
}

.stats {
  display: flex;
  gap: 15px;
  font-size: 16px;
}

.power-display {
  font-size: 18px;
  color: #ffd700;
}

.power-display strong {
  font-size: 22px;
  text-shadow: 0 0 10px rgba(255, 215, 0, 0.5);
}

.field {
  margin-bottom: 20px;
}

.field-label {
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 10px;
}

.field-grid {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 10px;
  padding: 0 80px;
}

.field-slot {
  background: rgba(255, 255, 255, 0.2);
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 8px;
  padding: 8px;
  min-height: 70px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
  transition: all 0.3s;
}

.field-slot.selectable {
  border-color: #4caf50;
  cursor: pointer;
  box-shadow: 0 0 15px rgba(76, 175, 80, 0.5);
}

.field-slot.selectable:hover {
  transform: scale(1.05);
  box-shadow: 0 0 20px rgba(76, 175, 80, 0.8);
}

.field-slot.selected {
  border-color: #ffd700;
  box-shadow: 0 0 20px rgba(255, 215, 0, 0.8);
}

.field-card {
  text-align: center;
  width: 100%;
}

.field-card.hidden {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
  border-radius: 8px;
}

.card-back {
  font-size: 48px;
  font-weight: bold;
}

.card-name-small {
  font-size: 12px;
  font-weight: bold;
  margin-bottom: 4px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 100%;
}

.card-power {
  font-size: 20px;
  font-weight: bold;
}

.empty-slot {
  color: rgba(255, 255, 255, 0.5);
  font-size: 14px;
}

.extra-slot-container {
  margin-top: 10px;
  padding-top: 10px;
  border-top: 1px dashed rgba(255, 255, 255, 0.3);
}

.extra-slot {
  background: rgba(255, 215, 0, 0.2);
  border: 1px dashed rgba(255, 215, 0, 0.5);
  border-radius: 4px;
  padding: 5px;
  margin-top: 5px;
  transition: all 0.3s;
}

.extra-slot.selectable {
  border-color: #4caf50;
  cursor: pointer;
}

.extra-slot.selectable:hover {
  transform: scale(1.05);
}

.field-card.extra {
  font-size: 12px;
}

.empty-slot.extra {
  font-size: 10px;
}

.hand {
  margin-bottom: 20px;
}

.hand-label {
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 10px;
}

.hint {
  color: #ff9800;
  font-size: 14px;
  font-weight: normal;
  margin-left: 10px;
}

.hint-disabled {
  color: #999;
  font-size: 14px;
  font-weight: normal;
  margin-left: 10px;
}

.hint-extra {
  color: #4caf50;
  font-size: 14px;
  font-weight: bold;
  margin-left: 10px;
  animation: pulse 1s infinite;
}

.hand-cards {
  display: flex;
  gap: 15px;
  overflow-x: auto;
  padding: 10px 0;
}

.hand-card {
  background: rgba(255, 255, 255, 0.95);
  color: #333;
  border-radius: 12px;
  padding: 15px;
  min-width: 200px;
  max-width: 200px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  transition: all 0.3s;
  border: 2px solid transparent;
  cursor: pointer;
}

.hand-card.playable {
  border-color: #4caf50;
  box-shadow: 0 0 15px rgba(76, 175, 80, 0.5);
}

.hand-card.playable:hover {
  transform: translateY(-10px);
  box-shadow: 0 10px 25px rgba(76, 175, 80, 0.7);
}

.hand-card.disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.hand-card.selectable {
  border-color: #ff9800;
}

.hand-card.selectable:hover {
  transform: translateY(-5px);
  box-shadow: 0 5px 20px rgba(255, 152, 0, 0.7);
}

.hand-card.selected {
  border-color: #f44336;
  background: rgba(244, 67, 54, 0.1);
  transform: translateY(-5px);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 12px;
}

.card-attribute {
  background: rgba(0, 0, 0, 0.1);
  padding: 2px 6px;
  border-radius: 4px;
  font-weight: bold;
}

.card-cost-power {
  display: flex;
  gap: 5px;
}

.card-type-badge {
  background: #ff9800;
  color: white;
  padding: 2px 6px;
  border-radius: 4px;
  font-weight: bold;
}

.card-name {
  font-weight: bold;
  font-size: 16px;
  text-align: center;
}

.card-keywords {
  font-size: 12px;
  color: #666;
  text-align: center;
}

.card-effect {
  font-size: 11px;
  color: #666;
  font-style: italic;
  line-height: 1.3;
  min-height: 40px;
}

.target-selection {
  background: rgba(0, 0, 0, 0.3);
  padding: 15px;
  border-radius: 10px;
  margin-bottom: 15px;
}

.target-label {
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 10px;
}

.target-cards {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.target-card {
  background: rgba(255, 255, 255, 0.9);
  color: #333;
  padding: 10px 15px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
  border: 2px solid transparent;
}

.target-card:hover {
  border-color: #4caf50;
  transform: scale(1.05);
  box-shadow: 0 0 15px rgba(76, 175, 80, 0.5);
}

.actions {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  justify-content: center;
}

.btn {
  padding: 12px 24px;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s;
}

.btn-primary {
  background: #4caf50;
  color: white;
}

.btn-primary:hover {
  background: #45a049;
  transform: scale(1.05);
}

.btn-secondary {
  background: #2196f3;
  color: white;
}

.btn-secondary:hover {
  background: #0b7dda;
  transform: scale(1.05);
}

.btn-small {
  padding: 8px 16px;
  font-size: 14px;
  background: #ff9800;
  color: white;
}

.btn-small:hover {
  background: #e68900;
}

.reforge-options {
  display: flex;
  gap: 10px;
  align-items: center;
  flex-wrap: wrap;
  background: rgba(0, 0, 0, 0.3);
  padding: 15px;
  border-radius: 10px;
  width: 100%;
}

.reforge-info {
  font-weight: bold;
  margin-right: 10px;
}

.warning {
  color: #f44336;
  font-size: 14px;
}
</style>
