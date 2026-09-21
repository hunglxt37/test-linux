<template>
  <div class="page">
    <!-- Header -->
    <header class="header">
      <div class="header-inner">
        <div class="logo">
          <div class="logo-icon">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/>
              <circle cx="9" cy="7" r="4"/>
              <path d="M23 21v-2a4 4 0 0 0-3-3.87"/>
              <path d="M16 3.13a4 4 0 0 1 0 7.75"/>
            </svg>
          </div>
          <div>
            <h1 class="logo-title">SmartGrocery</h1>
            <p class="logo-sub">Quản lý người dùng</p>
          </div>
        </div>
        <div class="header-stats">
          <div class="stat-chip">
            <span class="stat-dot"></span>
            <span>{{ users.length }} người dùng</span>
          </div>
        </div>
      </div>
    </header>

    <!-- Main Content -->
    <main class="main">
      <!-- Form Card -->
      <section class="card form-card">
        <div class="card-header">
          <div class="card-icon" :class="isEditing ? 'icon-edit' : 'icon-add'">
            <svg v-if="!isEditing" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/>
            </svg>
            <svg v-else viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"/>
              <path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"/>
            </svg>
          </div>
          <div>
            <h2 class="card-title">{{ isEditing ? 'Cập nhật người dùng' : 'Thêm người dùng mới' }}</h2>
            <p class="card-desc">{{ isEditing ? 'Chỉnh sửa thông tin người dùng bên dưới' : 'Điền đầy đủ thông tin để tạo tài khoản mới' }}</p>
          </div>
        </div>

        <form @submit.prevent="saveUser" class="form">
          <div class="form-fields">
            <div class="field-wrap">
              <label class="field-label">Họ tên</label>
              <div class="input-wrap">
                <svg class="input-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/>
                </svg>
                <input id="input-name" v-model="form.name" placeholder="Nhập họ tên..." required />
              </div>
            </div>

            <div class="field-wrap">
              <label class="field-label">Email</label>
              <div class="input-wrap">
                <svg class="input-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/><polyline points="22,6 12,13 2,6"/>
                </svg>
                <input id="input-email" v-model="form.email" type="email" placeholder="example@email.com" required />
              </div>
            </div>

            <div class="field-wrap field-wrap--sm">
              <label class="field-label">Tuổi</label>
              <div class="input-wrap">
                <svg class="input-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <rect x="3" y="4" width="18" height="18" rx="2" ry="2"/><line x1="16" y1="2" x2="16" y2="6"/><line x1="8" y1="2" x2="8" y2="6"/><line x1="3" y1="10" x2="21" y2="10"/>
                </svg>
                <input id="input-age" v-model.number="form.age" type="number" placeholder="0" min="0" max="150" required />
              </div>
            </div>
          </div>

          <div class="form-actions">
            <button id="btn-submit" type="submit" class="btn btn-primary" :class="{ 'btn-edit': isEditing }">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path v-if="!isEditing" d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11l5 5v11a2 2 0 0 1-2 2z"/><polyline v-if="!isEditing" points="17,21 17,13 7,13 7,21"/><polyline v-if="!isEditing" points="7,3 7,8 15,8"/>
                <path v-if="isEditing" d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"/><path v-if="isEditing" d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"/>
              </svg>
              {{ isEditing ? 'Cập nhật' : 'Thêm mới' }}
            </button>
            <button id="btn-cancel" v-if="isEditing" type="button" class="btn btn-ghost" @click="resetForm">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/>
              </svg>
              Hủy
            </button>
          </div>
        </form>
      </section>

      <!-- Table Card -->
      <section class="card table-card">
        <div class="card-header">
          <div class="card-icon icon-list">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <line x1="8" y1="6" x2="21" y2="6"/><line x1="8" y1="12" x2="21" y2="12"/><line x1="8" y1="18" x2="21" y2="18"/>
              <line x1="3" y1="6" x2="3.01" y2="6"/><line x1="3" y1="12" x2="3.01" y2="12"/><line x1="3" y1="18" x2="3.01" y2="18"/>
            </svg>
          </div>
          <div>
            <h2 class="card-title">Danh sách người dùng</h2>
            <p class="card-desc">Tổng cộng {{ users.length }} bản ghi</p>
          </div>
        </div>

        <div class="table-wrap">
          <table class="table">
            <thead>
              <tr>
                <th>ID</th>
                <th>Họ tên</th>
                <th>Email</th>
                <th>Tuổi</th>
                <th>Hành động</th>
              </tr>
            </thead>
            <tbody>
              <tr v-if="users.length === 0">
                <td colspan="5" class="empty-row">
                  <div class="empty-state">
                    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                      <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/>
                      <circle cx="9" cy="7" r="4"/>
                      <path d="M23 21v-2a4 4 0 0 0-3-3.87"/>
                      <path d="M16 3.13a4 4 0 0 1 0 7.75"/>
                    </svg>
                    <p>Chưa có người dùng nào</p>
                  </div>
                </td>
              </tr>
              <tr v-for="(user, index) in users" :key="user.id" :style="{ '--i': index }">
                <td>
                  <span class="id-badge">#{{ user.id }}</span>
                </td>
                <td>
                  <div class="user-cell">
                    <div class="avatar">{{ getInitial(user.name) }}</div>
                    <span class="user-name">{{ user.name }}</span>
                  </div>
                </td>
                <td>
                  <span class="email-text">{{ user.email }}</span>
                </td>
                <td>
                  <span class="age-badge">{{ user.age }} tuổi</span>
                </td>
                <td>
                  <div class="action-group">
                    <button :id="`btn-edit-${user.id}`" class="btn-icon btn-icon--edit" @click="editUser(user)" title="Chỉnh sửa">
                      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                        <path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"/>
                        <path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"/>
                      </svg>
                    </button>
                    <button :id="`btn-delete-${user.id}`" class="btn-icon btn-icon--delete" @click="deleteUser(user.id)" title="Xóa">
                      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                        <polyline points="3 6 5 6 21 6"/><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a1 1 0 0 1 1-1h4a1 1 0 0 1 1 1v2"/>
                      </svg>
                    </button>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>
    </main>

    <!-- Toast Notification -->
    <transition name="toast">
      <div v-if="toast.show" class="toast" :class="`toast--${toast.type}`">
        <svg v-if="toast.type === 'success'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <polyline points="20 6 9 17 4 12"/>
        </svg>
        <svg v-else viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12.01" y2="16"/>
        </svg>
        {{ toast.message }}
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const API_URL = '/api/users'

const users = ref([])
const isEditing = ref(false)
const form = ref({ id: null, name: '', email: '', age: null })
const toast = ref({ show: false, message: '', type: 'success' })

const showToast = (message, type = 'success') => {
  toast.value = { show: true, message, type }
  setTimeout(() => { toast.value.show = false }, 3000)
}

const getInitial = (name) => name ? name.charAt(0).toUpperCase() : '?'

const fetchUsers = async () => {
  try {
    const res = await fetch(API_URL)
    if (res.ok) users.value = await res.json()
  } catch (err) {
    console.error('Lỗi khi tải dữ liệu:', err)
  }
}

const saveUser = async () => {
  try {
    const method = isEditing.value ? 'PUT' : 'POST'
    const url = isEditing.value ? `${API_URL}/${form.value.id}` : API_URL

    const res = await fetch(url, {
      method,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name: form.value.name, email: form.value.email, age: form.value.age })
    })

    if (res.ok) {
      await fetchUsers()
      resetForm()
      showToast(isEditing.value ? 'Cập nhật thành công!' : 'Thêm người dùng thành công!')
    } else {
      showToast('Có lỗi xảy ra từ máy chủ!', 'error')
    }
  } catch (err) {
    showToast('Không thể kết nối đến máy chủ!', 'error')
  }
}

const deleteUser = async (id) => {
  if (!confirm('Bạn có chắc muốn xóa người dùng này không?')) return
  try {
    const res = await fetch(`${API_URL}/${id}`, { method: 'DELETE' })
    if (res.ok) {
      fetchUsers()
      showToast('Đã xóa người dùng thành công!')
    }
  } catch (err) {
    showToast('Lỗi khi xóa!', 'error')
  }
}

const editUser = (user) => {
  isEditing.value = true
  form.value = { ...user }
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

const resetForm = () => {
  isEditing.value = false
  form.value = { id: null, name: '', email: '', age: null }
}

onMounted(() => { fetchUsers() })
</script>

<style scoped>
/* ── Page Layout ── */
.page {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* ── Header ── */
.header {
  position: sticky;
  top: 0;
  z-index: 100;
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  background: rgba(10, 14, 26, 0.8);
  border-bottom: 1px solid var(--border);
}

.header-inner {
  max-width: 1100px;
  margin: 0 auto;
  padding: 16px 24px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.logo {
  display: flex;
  align-items: center;
  gap: 14px;
}

.logo-icon {
  width: 44px;
  height: 44px;
  border-radius: 12px;
  background: linear-gradient(135deg, var(--accent), #8b5cf6);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 0 20px var(--accent-glow);
  flex-shrink: 0;
}

.logo-icon svg {
  width: 22px;
  height: 22px;
  color: white;
}

.logo-title {
  font-size: 18px;
  font-weight: 700;
  color: var(--text-primary);
  line-height: 1.2;
}

.logo-sub {
  font-size: 12px;
  color: var(--text-muted);
  font-weight: 400;
}

.stat-chip {
  display: flex;
  align-items: center;
  gap: 8px;
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 999px;
  padding: 6px 14px;
  font-size: 13px;
  color: var(--text-secondary);
  font-weight: 500;
}

.stat-dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: var(--success);
  box-shadow: 0 0 6px var(--success);
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.6; transform: scale(0.85); }
}

/* ── Main ── */
.main {
  max-width: 1100px;
  margin: 0 auto;
  width: 100%;
  padding: 32px 24px 64px;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

/* ── Card ── */
.card {
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  overflow: hidden;
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  box-shadow: var(--shadow);
  transition: border-color 0.3s;
}

.card:hover {
  border-color: rgba(255,255,255,0.12);
}

.card-header {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 24px 28px;
  border-bottom: 1px solid var(--border);
}

.card-icon {
  width: 42px;
  height: 42px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.card-icon svg {
  width: 20px;
  height: 20px;
}

.icon-add {
  background: rgba(99, 102, 241, 0.15);
  color: var(--accent-light);
  border: 1px solid rgba(99, 102, 241, 0.25);
}

.icon-edit {
  background: rgba(245, 158, 11, 0.15);
  color: #fbbf24;
  border: 1px solid rgba(245, 158, 11, 0.25);
}

.icon-list {
  background: rgba(16, 185, 129, 0.12);
  color: #34d399;
  border: 1px solid rgba(16, 185, 129, 0.2);
}

.card-title {
  font-size: 16px;
  font-weight: 600;
  color: var(--text-primary);
  line-height: 1.3;
}

.card-desc {
  font-size: 13px;
  color: var(--text-muted);
  margin-top: 2px;
}

/* ── Form ── */
.form {
  padding: 24px 28px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-fields {
  display: grid;
  grid-template-columns: 1fr 1fr 140px;
  gap: 16px;
}

.field-label {
  display: block;
  font-size: 12px;
  font-weight: 600;
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 0.06em;
  margin-bottom: 8px;
}

.input-wrap {
  position: relative;
}

.input-icon {
  position: absolute;
  left: 12px;
  top: 50%;
  transform: translateY(-50%);
  width: 16px;
  height: 16px;
  color: var(--text-muted);
  pointer-events: none;
  transition: color 0.2s;
}

.input-wrap input {
  width: 100%;
  padding: 11px 14px 11px 38px;
  background: rgba(255,255,255,0.04);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  color: var(--text-primary);
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  outline: none;
  transition: all 0.2s;
}

.input-wrap input::placeholder {
  color: var(--text-muted);
}

.input-wrap input:focus {
  border-color: var(--accent);
  background: rgba(99, 102, 241, 0.06);
  box-shadow: 0 0 0 3px var(--accent-glow);
}

.input-wrap input:focus + .input-icon,
.input-wrap:focus-within .input-icon {
  color: var(--accent-light);
}

/* Fix icon z-order */
.input-wrap .input-icon {
  z-index: 1;
}

.form-actions {
  display: flex;
  gap: 12px;
}

/* ── Buttons ── */
.btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 11px 22px;
  border-radius: var(--radius);
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  border: none;
  transition: all 0.2s;
}

.btn svg {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
}

.btn-primary {
  background: linear-gradient(135deg, var(--accent), #8b5cf6);
  color: white;
  box-shadow: 0 4px 14px var(--accent-glow);
}

.btn-primary:hover {
  transform: translateY(-1px);
  box-shadow: 0 6px 20px rgba(99,102,241,0.4);
}

.btn-primary:active {
  transform: translateY(0);
}

.btn-edit {
  background: linear-gradient(135deg, #f59e0b, #d97706);
  box-shadow: 0 4px 14px rgba(245,158,11,0.3);
}

.btn-edit:hover {
  box-shadow: 0 6px 20px rgba(245,158,11,0.4);
}

.btn-ghost {
  background: transparent;
  border: 1px solid var(--border);
  color: var(--text-secondary);
}

.btn-ghost:hover {
  background: var(--bg-card-hover);
  color: var(--text-primary);
  border-color: rgba(255,255,255,0.15);
}

/* ── Table ── */
.table-wrap {
  overflow-x: auto;
}

.table {
  width: 100%;
  border-collapse: collapse;
}

.table thead tr {
  border-bottom: 1px solid var(--border);
}

.table th {
  padding: 14px 20px;
  text-align: left;
  font-size: 11px;
  font-weight: 700;
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 0.07em;
}

.table tbody tr {
  border-bottom: 1px solid rgba(255,255,255,0.04);
  transition: background 0.2s;
  animation: rowIn 0.3s ease both;
  animation-delay: calc(var(--i) * 0.05s);
}

@keyframes rowIn {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}

.table tbody tr:last-child { border-bottom: none; }

.table tbody tr:hover {
  background: var(--bg-card-hover);
}

.table td {
  padding: 14px 20px;
  font-size: 14px;
  color: var(--text-secondary);
  vertical-align: middle;
}

.id-badge {
  font-size: 12px;
  font-weight: 700;
  color: var(--text-muted);
  font-family: 'Courier New', monospace;
}

.user-cell {
  display: flex;
  align-items: center;
  gap: 10px;
}

.avatar {
  width: 34px;
  height: 34px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--accent), #8b5cf6);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 13px;
  font-weight: 700;
  color: white;
  flex-shrink: 0;
}

.user-name {
  font-weight: 500;
  color: var(--text-primary);
}

.email-text {
  font-size: 13px;
  color: var(--text-muted);
}

.age-badge {
  display: inline-flex;
  align-items: center;
  background: rgba(99,102,241,0.1);
  color: var(--accent-light);
  border: 1px solid rgba(99,102,241,0.2);
  border-radius: 999px;
  padding: 3px 10px;
  font-size: 12px;
  font-weight: 600;
}

.action-group {
  display: flex;
  gap: 8px;
}

.btn-icon {
  width: 34px;
  height: 34px;
  border-radius: 8px;
  border: none;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.btn-icon svg {
  width: 15px;
  height: 15px;
}

.btn-icon--edit {
  background: rgba(99,102,241,0.12);
  color: var(--accent-light);
  border: 1px solid rgba(99,102,241,0.2);
}

.btn-icon--edit:hover {
  background: rgba(99,102,241,0.25);
  transform: scale(1.08);
  box-shadow: 0 0 10px rgba(99,102,241,0.3);
}

.btn-icon--delete {
  background: var(--danger-glow);
  color: var(--danger-light);
  border: 1px solid rgba(239,68,68,0.2);
}

.btn-icon--delete:hover {
  background: rgba(239,68,68,0.3);
  transform: scale(1.08);
  box-shadow: 0 0 10px rgba(239,68,68,0.3);
}

/* ── Empty State ── */
.empty-row { text-align: center; }

.empty-state {
  padding: 48px 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  color: var(--text-muted);
}

.empty-state svg {
  width: 48px;
  height: 48px;
  opacity: 0.4;
}

.empty-state p {
  font-size: 14px;
}

/* ── Toast ── */
.toast {
  position: fixed;
  bottom: 28px;
  right: 28px;
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 14px 20px;
  border-radius: var(--radius);
  font-size: 14px;
  font-weight: 500;
  z-index: 999;
  backdrop-filter: blur(12px);
  box-shadow: var(--shadow-lg);
}

.toast svg {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
}

.toast--success {
  background: rgba(16,185,129,0.15);
  border: 1px solid rgba(16,185,129,0.3);
  color: #34d399;
}

.toast--error {
  background: rgba(239,68,68,0.15);
  border: 1px solid rgba(239,68,68,0.3);
  color: var(--danger-light);
}

.toast-enter-active, .toast-leave-active {
  transition: all 0.35s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.toast-enter-from, .toast-leave-to {
  opacity: 0;
  transform: translateY(16px) scale(0.94);
}

/* ── Responsive ── */
@media (max-width: 768px) {
  .header-inner { padding: 14px 16px; }
  .main { padding: 20px 16px 48px; }
  .form-fields { grid-template-columns: 1fr; }
  .card-header { padding: 20px 20px; }
  .form { padding: 20px; }
  .table th, .table td { padding: 12px 14px; }
  .logo-sub { display: none; }
}
</style>