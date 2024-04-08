<template>
  <div class="rootWrapper">
    <h1 class="title">Tính điểm team</h1>
    <div class="sub-title">by @tqcong</div>

    <div class="team-wrapper" v-if="selectedItem">
      <div class="team-title-wrapper">
        <input ref="teamNameInputRef" class="team-name-input" type="text" v-model="selectedItem.name"
          placeholder="Nhập tên đội" />
        <button class="delete-team-button button" @click="deleteTeam(selectedItem.id)">
          Xóa đội
        </button>
      </div>

      <div class="list-score">
        <div class="score-item" v-for="(_, index) in selectedItem.scores">
          <input ref="scoreInputRef" class="score-input" type="number" @keydown="handleKeyDownScoreBox"
            v-model="selectedItem.scores[index]" />
          <button class="delete-score-button button button-icon" @click="deleteScore(selectedItem.id, index)">
            &times;
          </button>
        </div>
        <button class="add-score-button button-icon" @click="addScore(selectedItem)">
          +
        </button>
      </div>

      <div class="summary-wrapper">
        <div class="label">Tổng:&nbsp;</div>
        <div class="total-value">{{ getTotalScore(selectedItem) }}</div>
      </div>
    </div>
    <div v-else :class="{
      buttons: true,
      emptyItemState: !selectedItem
    }">
      <button id="addTeamButton" class="button-icon" @click="addTeam()">
        + Thêm đội
      </button>
    </div>

    <nav class="navbar">
      <div class="list">
        <div :class="[
      'list-item col-4',
      {
        active: team.id === selectedItem?.id,
      },
    ]" v-for="team in teams" :key="team.id" @click="switchTab(team)">
          <div class="child">
            <div class="team-name" :title="team.name">
              {{ team.name || 'Đội mới' }}
            </div>
            <div class="team-score">({{ getTotalScore(team) }})</div>
          </div>
        </div>
      </div>

      <div class="buttons">
        <button id="addTeamButton" class="button-icon" @click="addTeam()">
          +
        </button>
      </div>
      <hr />
      <div class="action-buttons">
        <button id="showResultButton" @click="showResult">Xem kết quả</button>
        <button id="clearAllButton" @click="clearAll">Xóa hết</button>
      </div>
    </nav>

    <!-- popup result -->
    <div v-if="hasShowResult" class="popup-overlay" ref="popupResultRef">
      <div class="popup-result">
        <div class="popup-header">
          <div class="title">Kết quả</div>
          <button @click="hasShowResult = false">&times;</button>
        </div>
        <div class="popup-content">
          <template v-if="results.length > 0">
            <div class="result-item" v-for="result in results">
              <div class="result-item-top">Top&nbsp;{{ result.top }}:</div>
              <div class="result-item-name">{{ result.name }}&nbsp;-</div>
              <div class="result-item-score">{{ result.score }}&nbsp;điểm</div>
            </div>
          </template>
          <template v-else>Không có dữ liệu.</template>
        </div>
        <div class="popup-footer">
          <button @click="hasShowResult = false">Đóng</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick, onMounted } from 'vue';
import { localStorageUtil } from '../utils/local-storage-util.js';

const teams = ref([]);
const lsKey = 'tqcong_items';

onMounted(() => {
  const itemsFromLs = localStorageUtil.getItem(lsKey);
  if (itemsFromLs) {
    teams.value = itemsFromLs;
  }
  selectedItem.value = getFirstItem();

  window.addEventListener('beforeunload', handleUnloadPage);
});

function handleUnloadPage() {
  localStorageUtil.setItem(lsKey, teams.value);
  window.removeEventListener('beforeunload', handleUnloadPage);
}

const teamNameInputRef = ref([]);

const scoreInputRef = ref([]);

function getTotalScore(team) {
  if (!team) return 0;
  const scores = team.scores;
  const total = scores.reduce(
    (accumulator, currentValue) => accumulator + currentValue,
    0
  );
  return total;
}

async function addTeam() {
  const id = new Date().getTime();
  const newTeam = {
    id: id,
    name: '',
    scores: [null],
  };
  teams.value.push(newTeam);

  selectedItem.value = newTeam;

  await nextTick();
  teamNameInputRef.value.focus();
}

function deleteTeam(id) {
  const index = teams.value.findIndex((team) => team.id === id);
  if (index != -1) {
    teams.value.splice(index, 1);

    selectedItem.value = getFirstItem();
  }
}

function getFirstItem() {
  return teams.value[teams.value.length - 1];
}

async function addScore(team) {
  team.scores.push(0);
  await nextTick();

  const lastInput = scoreInputRef.value[scoreInputRef.value.length - 1];
  lastInput.value = null;
  lastInput.focus();
}

function deleteScore(id, scoreIndex) {
  const index = teams.value.findIndex((team) => team.id === id);
  if (index != -1) {
    const team = teams.value[index];
    const scores = team.scores;
    scores.splice(scoreIndex, 1);
  }
}

const selectedItem = ref(null);

function switchTab(team, hasFocus = false) {
  selectedItem.value = team;

  if (hasFocus) {
    teamNameInputRef.value.focus();
  }
}

async function clearAll() {
  teams.value = [];
  selectedItem.value = null;
  results.value = [];
  await nextTick();
}

const hasShowResult = ref(false);

const results = ref([]);

function getResults() {
  const tmpTeams = [...teams.value];
  tmpTeams.forEach((team) => {
    const totalScore = getTotalScore(team);
    team.totalScore = totalScore;
  });

  tmpTeams.sort((a, b) => b.totalScore - a.totalScore);

  const results = [];
  tmpTeams.forEach((team, index) => {
    const result = {
      name: team.name,
      top: index + 1,
      score: team.totalScore,
    };
    results.push(result);
  });

  return results;
}

const popupResultRef = ref(null);
async function showResult() {
  results.value = getResults();
  hasShowResult.value = true;
  await nextTick();

  popupResultRef.value.focus();
}

function handleKeyDownScoreBox(event) {
  if (event.key == 'Enter') {
    event.stopPropagation();
    addScore(selectedItem.value)
  }
}
</script>

<style scoped>
.rootWrapper {
  display: flex;
  flex-direction: column;
  width: 100vw;
  height: 100vh;
}

.team-wrapper {
  border-radius: 4px;
  padding: 12px;

  display: flex;
  flex-direction: column;
  gap: 12px;

  flex: 1;

  overflow-y: scroll;
}

.team-title-wrapper {
  display: flex;
  gap: 8px;
}

.team-title-wrapper .team-name-input {
  font-size: 1.2rem;
  font-weight: bold;
  max-width: 100%;
  padding: 8px;
}

.list-score {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 8px;
}

.list-score .score-item {
  display: flex;
  gap: 8px;
}

.list-score .score-input {
  width: 100px;
}

.summary-wrapper {
  display: flex;
}

.summary-wrapper .label {
  font-weight: 500;
}

.summary-wrapper .total-value {
  font-weight: bold;
}

.navbar {
  width: 100%;
  padding: 12px;
  background-color: #333;
  color: #fff;
}

.navbar .list {
  display: flex;
  margin-bottom: 8px;
  overflow-x: scroll;
}

.navbar .list-item {
  cursor: pointer;
  display: flex;
  flex-direction: column;
  flex-basis: 25%;
  border-radius: 4px;
}

.navbar .list-item .team-name {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.navbar .list-item .team-score {
  font-weight: bold;
}

.navbar .list-item.active {
  background: #ffd23f;
  color: #333;
}

.child {
  padding: 4px 8px;
}

.sub-title {
  font-size: 0.8rem;
  color: #777;
  text-align: center;
  margin-bottom: 24px;
}

h1.title {
  font-size: 1.8em;
  margin-block-start: 0.5em;
  margin-block-end: 0.25em;
  text-align: center;
}

.action-buttons {
  display: flex;
  gap: 16px;
}

/**popup */
.result-item {
  display: flex;
  gap: 8px;
  font-weight: 500;
  margin-bottom: 6px;
}

.popup-overlay {
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: rgba(0, 0, 0, 0.35);
  z-index: 999;
  display: flex;
  justify-content: center;
  align-items: center;
  transform: translateY(-10%);
}

.popup-result {
  box-shadow: rgba(60, 64, 67, 0.3) 0px 1px 2px 0px,
    rgba(60, 64, 67, 0.15) 0px 1px 3px 1px;
  padding: 24px;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  row-gap: 16px;
  background-color: #fff;
  width: 80vw;
  max-width: 480px;
}

.popup-header {
  display: flex;
  justify-content: space-between;
}

.popup-header .title {
  font-size: 1.4rem;
  font-weight: bold;
}

.popup-footer {
  display: flex;
  justify-content: flex-end;
}

.button-icon {
  font-size: 1.2rem;
}

.emptyItemState {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  transform: translateY(-20%);
}
</style>
