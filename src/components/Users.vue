<script setup lang="ts">
import { ref, computed } from 'vue'
import rawUsersData from '../data/users.json'
import type { User } from '../types/user'

const users = ref<User[]>(
  (rawUsersData as User[]).map(user => ({
    ...user,
    isDetailsVisible: false
  }))
)

const genderFilter = ref<'all' | 'male' | 'female'>('all')
const isAdultOnly = ref<boolean>(false)
const sortType = ref<'default' | 'name-asc' | 'name-desc' | 'age-asc' | 'age-desc'>('default')

const toggleDetails = (userId: number): void => {
  const target = users.value.find(u => u.id === userId)
  if (target) {
    target.isDetailsVisible = !target.isDetailsVisible
  }
}

const resetFilters = (): void => {
  genderFilter.value = 'all'
  isAdultOnly.value = false
  sortType.value = 'default'
}

const filteredUsers = computed(() => {
  let result = [...users.value]

  if (genderFilter.value !== 'all') {
    result = result.filter(u => u.gender === genderFilter.value)
  }

  if (isAdultOnly.value) {
    result = result.filter(u => u.dob.age >= 18)
  }

  switch (sortType.value) {
    case 'name-asc':
      result.sort((a, b) => a.name.first.localeCompare(b.name.first))
      break
    case 'name-desc':
      result.sort((a, b) => b.name.first.localeCompare(a.name.first))
      break
    case 'age-asc':
      result.sort((a, b) => a.dob.age - b.dob.age)
      break
    case 'age-desc':
      result.sort((a, b) => b.dob.age - a.dob.age)
      break
  }

  return result
})
</script>

<template>
  <div class="users-container">
    <div class="toolbar">
      <div class="control-group">
        <span class="group-label">Стать:</span>
        <button :class="{ active: genderFilter === 'all' }" @click="genderFilter = 'all'">Всі</button>
        <button :class="{ active: genderFilter === 'male' }" @click="genderFilter = 'male'">Чоловіки</button>
        <button :class="{ active: genderFilter === 'female' }" @click="genderFilter = 'female'">Жінки</button>
      </div>

      <div class="control-group">
        <span class="group-label">Вік:</span>
        <button :class="{ active: !isAdultOnly }" @click="isAdultOnly = false">Всі</button>
        <button :class="{ active: isAdultOnly }" @click="isAdultOnly = true">18+</button>
      </div>

      <div class="control-group">
        <span class="group-label">Сортування:</span>
        <button :class="{ active: sortType === 'name-asc' }" @click="sortType = 'name-asc'">Ім'я ↑</button>
        <button :class="{ active: sortType === 'name-desc' }" @click="sortType = 'name-desc'">Ім'я ↓</button>
        <button :class="{ active: sortType === 'age-asc' }" @click="sortType = 'age-asc'">Вік ↑</button>
        <button :class="{ active: sortType === 'age-desc' }" @click="sortType = 'age-desc'">Вік ↓</button>
      </div>

      <div class="control-group">
        <button class="btn-reset" @click="resetFilters">Очистити все</button>
      </div>
    </div>

    <p v-if="filteredUsers.length === 0" class="empty-state">
      Список юзерів пустий
    </p>

    <div v-else class="cards-grid">
      <div
        v-for="user in filteredUsers"
        :key="user.id"
        class="user-card"
        :class="{
          minor: user.dob.age < 18,
          young: user.dob.age >= 18 && user.dob.age <= 30,
          adult: user.dob.age >= 31 && user.dob.age <= 50,
          senior: user.dob.age > 50
        }"
      >
        <img
          :src="user.picture"
          :alt="`${user.name.first} ${user.name.last}`"
          class="user-avatar"
        />

        <div class="user-info">
          <h3>{{ user.name.title }} {{ user.name.first }} {{ user.name.last }}</h3>
          <p><strong>Email:</strong> {{ user.email }}</p>
          <p><strong>Телефон:</strong> {{ user.phone }}</p>
          <p><strong>Місто:</strong> {{ user.location.city }}, {{ user.location.country }}</p>

          <p v-if="user.dob.age > 18">
            <strong>Вік:</strong> {{ user.dob.age }}
          </p>

          <div class="hobbies-block">
            <strong>Хобі:</strong>
            <ul>
              <li v-for="(hobby, idx) in user.hobbies" :key="idx">
                {{ hobby }}
              </li>
            </ul>
          </div>

          <button class="btn-toggle" @click="toggleDetails(user.id)">
            {{ user.isDetailsVisible ? 'Сховати деталі' : 'Показати деталі' }}
          </button>

          <div v-show="user.isDetailsVisible" class="user-details">
            {{ user.details }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.users-container {
  max-width: 1100px;
  margin: 0 auto;
  padding: 24px 16px;
  font-family: system-ui, sans-serif;
}

.toolbar {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  background-color: #f8fafc;
  padding: 16px;
  border-radius: 10px;
  border: 1px solid #e2e8f0;
  margin-bottom: 24px;
}

.control-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.group-label {
  font-weight: 600;
  font-size: 14px;
}

button {
  padding: 6px 12px;
  background-color: #ffffff;
  border: 1px solid #cbd5e1;
  border-radius: 6px;
  cursor: pointer;
  font-size: 13px;
}

button.active {
  background-color: #3b82f6;
  color: #ffffff;
  border-color: #2563eb;
}

.btn-reset {
  background-color: #fee2e2;
  border-color: #fca5a5;
  color: #b91c1c;
}

.empty-state {
  text-align: center;
  font-size: 18px;
  color: #64748b;
  margin-top: 40px;
}

.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px;
}

.user-card {
  border-radius: 10px;
  padding: 16px;
  border-left: 8px solid transparent;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
  background-color: #ffffff;
}

.user-card.minor {
  border-left-color: #ef4444;
  background: #fef2f2;
}

.user-card.young {
  border-left-color: #10b981;
  background: #ecfdf5;
}

.user-card.adult {
  border-left-color: #3b82f6;
  background: #eff6ff;
}

.user-card.senior {
  border-left-color: #8b5cf6;
  background: #f5f3ff;
}

.user-avatar {
  width: 80px;
  height: 80px;
  object-fit: cover;
  border-radius: 50%;
  margin-bottom: 12px;
}

.user-info h3 {
  margin: 0 0 8px 0;
}

.user-info p {
  margin: 4px 0;
  font-size: 14px;
}

.hobbies-block {
  margin-top: 8px;
  font-size: 14px;
}

.hobbies-block ul {
  margin: 4px 0;
  padding-left: 20px;
}

.btn-toggle {
  margin-top: 10px;
  width: 100%;
}

.user-details {
  margin-top: 10px;
  padding: 8px;
  background-color: rgba(255, 255, 255, 0.7);
  border-radius: 6px;
  font-size: 13px;
  border: 1px dashed #cbd5e1;
}
</style>