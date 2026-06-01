<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 font-sans">
    <!-- Header -->
    <header class="bg-gradient-to-r from-blue-700 to-indigo-700 text-white shadow-lg">
      <div class="max-w-2xl mx-auto px-4 py-5 text-center">
        <div class="flex items-center justify-center gap-3 mb-1">
          <span class="text-4xl">🇹🇭</span>
          <h1 class="text-2xl font-bold tracking-wide">ไทยช่วยไทยพลัส</h1>
        </div>
        <p class="text-blue-200 text-sm">คำนวณส่วนลดค่าอาหารและค่าครองชีพ</p>
      </div>
    </header>

    <main class="max-w-2xl mx-auto px-4 py-8 space-y-6">
      <!-- Info Card -->
      <div class="bg-white rounded-2xl shadow p-5 border-l-4 border-blue-500">
        <h2 class="font-bold text-gray-700 mb-3 flex items-center gap-2">
          <span class="text-blue-500">ℹ️</span> เงื่อนไขโครงการ
        </h2>
        <div class="grid grid-cols-3 gap-3 text-center">
          <div class="bg-blue-50 rounded-xl p-3">
            <div class="text-2xl font-bold text-blue-600">60%</div>
            <div class="text-xs text-gray-500 mt-1">รัฐบาลช่วยจ่าย</div>
          </div>
          <div class="bg-orange-50 rounded-xl p-3">
            <div class="text-2xl font-bold text-orange-500">40%</div>
            <div class="text-xs text-gray-500 mt-1">ประชาชนจ่ายเอง</div>
          </div>
          <div class="bg-green-50 rounded-xl p-3">
            <div class="text-2xl font-bold text-green-600">200 ฿</div>
            <div class="text-xs text-gray-500 mt-1">เพดานรัฐ/วัน</div>
          </div>
        </div>
      </div>

      <!-- Calculator Card -->
      <div class="bg-white rounded-2xl shadow p-6">
        <h2 class="font-bold text-gray-700 mb-4 text-lg flex items-center gap-2">
          <span>🧮</span> คำนวณราคา
        </h2>

        <!-- Input -->
        <div class="mb-6">
          <label class="block text-sm text-gray-500 mb-2 font-medium">ราคาสินค้า / อาหาร (บาท)</label>
          <div class="relative">
            <input
              v-model="expression"
              type="text"
              inputmode="decimal"
              placeholder="เช่น 250  หรือ  150 + 80 - 30"
              :class="[
                'w-full border-2 rounded-xl px-4 py-3 pr-14 text-xl font-semibold text-gray-700 focus:outline-none transition',
                exprError ? 'border-red-300 focus:border-red-400' : 'border-gray-200 focus:border-blue-400'
              ]"
            />
            <span class="absolute right-4 top-1/2 -translate-y-1/2 text-gray-400 font-medium">฿</span>
          </div>
          <!-- Evaluated value hint -->
          <div class="mt-1.5 h-5 text-sm">
            <span v-if="isExpr && price > 0" class="text-blue-500 font-medium">
              = {{ formatBaht(price) }}
            </span>
            <span v-else-if="exprError" class="text-red-400">รูปแบบไม่ถูกต้อง</span>
          </div>
        </div>

        <!-- Quick preset buttons -->
        <div class="flex flex-wrap gap-2 mb-6">
          <span class="text-xs text-gray-400 self-center">ตัวอย่าง:</span>
          <button
            v-for="preset in presets"
            :key="preset"
            @click="expression = String(preset)"
            class="px-3 py-1.5 text-sm rounded-lg border border-gray-200 hover:border-blue-400 hover:bg-blue-50 hover:text-blue-600 transition text-gray-600"
          >
            {{ preset.toLocaleString() }} ฿
          </button>
        </div>

        <!-- Result -->
        <div v-if="price > 0" class="space-y-4">
          <!-- Case banner -->
          <div
            :class="isOverCap ? 'bg-amber-50 border-amber-400 text-amber-700' : 'bg-green-50 border-green-400 text-green-700'"
            class="border-l-4 rounded-xl px-4 py-3 text-sm font-medium flex items-center gap-2"
          >
            <span>{{ isOverCap ? '⚠️' : '✅' }}</span>
            <span>{{ isOverCap ? `รัฐช่วยเต็มเพดาน ${GOV_CAP} บาท (ไม่เกิน 60% เพราะเกินเพดาน)` : 'รัฐช่วยจ่าย 60% ปกติ (ยังไม่เกินเพดาน)' }}</span>
          </div>

          <!-- Breakdown -->
          <div class="bg-gray-50 rounded-xl p-4 space-y-3">
            <div class="flex justify-between items-center">
              <div class="flex items-center gap-2">
                <div class="w-3 h-3 rounded-full bg-gray-400"></div>
                <span class="text-gray-600 text-sm">ราคาสินค้าทั้งหมด</span>
              </div>
              <span class="font-bold text-gray-700">{{ formatBaht(price) }}</span>
            </div>

            <div class="border-t border-gray-200 pt-3 flex justify-between items-center">
              <div class="flex items-center gap-2">
                <div class="w-3 h-3 rounded-full bg-blue-500"></div>
                <span class="text-blue-600 text-sm">
                  รัฐช่วยจ่าย {{ isOverCap ? `(เพดาน ${GOV_CAP} ฿)` : '(60%)' }}
                </span>
              </div>
              <span class="font-bold text-blue-600 text-lg">{{ formatBaht(govAmount) }}</span>
            </div>

            <div class="flex justify-between items-center">
              <div class="flex items-center gap-2">
                <div class="w-3 h-3 rounded-full bg-orange-500"></div>
                <span class="text-orange-600 text-sm">ประชาชนจ่ายเอง {{ isOverCap ? '' : '(40%)' }}</span>
              </div>
              <span class="font-bold text-orange-500 text-lg">{{ formatBaht(userAmount) }}</span>
            </div>
          </div>

          <!-- Visual bar -->
          <div>
            <div class="flex rounded-full overflow-hidden h-6 shadow-inner">
              <div
                class="bg-blue-500 flex items-center justify-center text-white text-xs font-bold transition-all duration-500"
                :style="{ width: govPercent + '%' }"
              >
                {{ govPercent.toFixed(0) }}%
              </div>
              <div
                class="bg-orange-400 flex items-center justify-center text-white text-xs font-bold transition-all duration-500"
                :style="{ width: (100 - govPercent) + '%' }"
              >
                {{ (100 - govPercent).toFixed(0) }}%
              </div>
            </div>
            <div class="flex justify-between mt-1 text-xs text-gray-400">
              <span>🏛️ รัฐบาล</span>
              <span>👤 ประชาชน</span>
            </div>
          </div>

          <!-- Summary box -->
          <div class="bg-gradient-to-r from-blue-600 to-indigo-600 rounded-2xl p-5 text-white">
            <p class="text-blue-200 text-sm mb-1">💳 ยอดที่ต้องกดจ่ายในแอปเป๋าตัง (G-Wallet)</p>
            <p class="text-4xl font-extrabold mb-3">{{ formatBaht(userAmount) }}</p>
            <div class="bg-white/20 rounded-xl p-3 text-sm space-y-1">
              <div class="flex justify-between">
                <span class="text-blue-200">โควตารัฐฯ ถูกใช้วันนี้</span>
                <span class="font-semibold">{{ formatBaht(govAmount) }}</span>
              </div>
              <div class="flex justify-between">
                <span class="text-blue-200">โควตาคงเหลือ (ถ้าใช้ครั้งเดียว)</span>
                <span class="font-semibold">{{ formatBaht(GOV_CAP - govAmount) }}</span>
              </div>
            </div>
          </div>

          <!-- Remaining quota bar -->
          <div v-if="govAmount < GOV_CAP" class="bg-white border border-gray-200 rounded-2xl p-4">
            <h3 class="text-sm font-bold text-gray-600 mb-3 flex items-center gap-1">
              <span>📊</span> โควตารัฐที่เหลือวันนี้
            </h3>
            <div class="w-full bg-gray-100 rounded-full h-4 overflow-hidden">
              <div
                class="bg-green-400 h-4 rounded-full transition-all duration-500"
                :style="{ width: ((GOV_CAP - govAmount) / GOV_CAP * 100) + '%' }"
              ></div>
            </div>
            <div class="flex justify-between mt-2 text-xs text-gray-500">
              <span>ใช้ไป {{ formatBaht(govAmount) }}</span>
              <span>คงเหลือ {{ formatBaht(GOV_CAP - govAmount) }}</span>
            </div>
          </div>
        </div>

        <!-- Empty state -->
        <div v-else class="text-center py-8 text-gray-300">
          <div class="text-5xl mb-3">🛒</div>
          <p class="text-sm">กรอกราคาสินค้าเพื่อคำนวณ</p>
        </div>
      </div>

      <!-- Examples Section -->
      <div class="bg-white rounded-2xl shadow p-6">
        <h2 class="font-bold text-gray-700 mb-4 flex items-center gap-2">
          <span>📖</span> ตัวอย่างการคำนวณ
        </h2>
        <div class="space-y-4">
          <div
            v-for="ex in examples"
            :key="ex.price"
            class="border border-gray-100 rounded-xl p-4 hover:border-blue-200 transition cursor-pointer"
            @click="expression = String(ex.price)"
          >
            <div class="flex justify-between items-start mb-2">
              <div>
                <span class="font-semibold text-gray-700">{{ ex.label }}</span>
                <span class="ml-2 text-sm text-gray-400">{{ formatBaht(ex.price) }}</span>
              </div>
              <span
                :class="ex.over ? 'bg-amber-100 text-amber-600' : 'bg-green-100 text-green-600'"
                class="text-xs px-2 py-0.5 rounded-full font-medium"
              >
                {{ ex.over ? 'เกินเพดาน' : 'ปกติ' }}
              </span>
            </div>
            <div class="flex gap-4 text-sm">
              <span class="text-blue-600">🏛️ รัฐจ่าย {{ formatBaht(ex.gov) }}</span>
              <span class="text-orange-500">👤 จ่ายเอง {{ formatBaht(ex.user) }}</span>
            </div>
          </div>
        </div>
      </div>

      <p class="text-center text-xs text-gray-400 pb-4">
        ข้อมูลอ้างอิงจากโครงการไทยช่วยไทยพลัส • คำนวณเพื่อประกอบการตัดสินใจเท่านั้น
      </p>
    </main>
  </div>
</template>

<script setup lang="ts">
useSeoMeta({
  title: 'คำนวณไทยช่วยไทยพลัส — รู้ก่อนจ่าย รัฐออกให้เท่าไหร่?',
  description: 'คำนวณราคาอาหารและสินค้าจากโครงการไทยช่วยไทยพลัส รัฐบาลออกให้ 60% สูงสุด 200 บาท/วัน รู้ทันทีว่าต้องจ่ายเองเท่าไหร่ผ่านแอปเป๋าตัง',
  ogTitle: 'คำนวณไทยช่วยไทยพลัส — รู้ก่อนจ่าย',
  ogDescription: 'รัฐออกให้ 60% (สูงสุด 200 ฿/วัน) คุณออกเอง 40% กรอกราคาแล้วรู้เลยว่าต้องสแกนจ่ายในเป๋าตังเท่าไหร่',
  ogType: 'website',
  twitterCard: 'summary',
  twitterTitle: 'คำนวณไทยช่วยไทยพลัส',
  twitterDescription: 'รัฐออกให้ 60% สูงสุด 200 ฿/วัน กรอกราคาแล้วรู้เลยว่าต้องจ่ายเองเท่าไหร่',
})

useHead({
  htmlAttrs: { lang: 'th' },
  link: [{ rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }],
})

const GOV_CAP = 200
const GOV_RATIO = 0.6

const expression = ref('')

function evalExpression(expr: string): number | null {
  const cleaned = expr.trim()
  if (!cleaned) return null
  if (!/^[\d\s\+\-\*\/\.]+$/.test(cleaned)) return null
  try {
    const result = new Function('return ' + cleaned)()
    if (typeof result !== 'number' || !isFinite(result) || result < 0) return null
    return result
  } catch {
    return null
  }
}

const price = computed(() => evalExpression(expression.value) ?? 0)

const isExpr = computed(() => /[\+\-\*\/]/.test(expression.value))
const exprError = computed(() => expression.value.trim() !== '' && evalExpression(expression.value) === null)

const govAmount = computed(() => {
  if (!price.value || price.value <= 0) return 0
  return Math.min(price.value * GOV_RATIO, GOV_CAP)
})

const userAmount = computed(() => {
  if (!price.value || price.value <= 0) return 0
  return price.value - govAmount.value
})

const isOverCap = computed(() => price.value * GOV_RATIO > GOV_CAP)

const govPercent = computed(() => {
  if (!price.value || price.value <= 0) return 0
  return (govAmount.value / price.value) * 100
})

const presets = [100, 200, 250, 333, 500, 750, 1000]

const examples = computed(() => {
  const cases = [
    { price: 250, label: 'อาหารกลางวัน' },
    { price: 333, label: 'อาหาร (เพดานพอดี 333.33 ฿)' },
    { price: 500, label: 'ของใช้ในบ้าน' },
    { price: 1000, label: 'ของชำชุดใหญ่' },
  ]
  return cases.map(c => {
    const gov = Math.min(c.price * GOV_RATIO, GOV_CAP)
    return {
      ...c,
      gov,
      user: c.price - gov,
      over: c.price * GOV_RATIO > GOV_CAP,
    }
  })
})

function formatBaht(amount: number): string {
  return amount.toLocaleString('th-TH', { minimumFractionDigits: 0, maximumFractionDigits: 2 }) + ' ฿'
}
</script>
