<template>
  <div class="min-h-screen p-4 flex flex-col gap-4 max-w-6xl mx-auto">
    <h1 class="text-3xl font-bold text-purple-400">盲文翻译与触觉学习器</h1>

    <div class="flex gap-2">
      <button v-for="t in tabs" :key="t.id" @click="activeTab = t.id"
        class="px-4 py-2 rounded text-sm"
        :class="activeTab === t.id ? 'bg-purple-500 text-white' : 'bg-gray-800 text-gray-300 hover:bg-gray-700'">
        {{ t.label }}
      </button>
    </div>

    <!-- Translate -->
    <div v-if="activeTab === 'translate'" class="grid grid-cols-2 gap-4">
      <div class="bg-gray-900 rounded-xl p-4">
        <h3 class="text-purple-300 font-bold mb-2">文本输入</h3>
        <textarea v-model="store.inputText" @input="store.translate()"
          class="w-full h-32 bg-gray-800 rounded p-3 text-white resize-none" placeholder="输入英文文本..." />
      </div>
      <div class="bg-gray-900 rounded-xl p-4">
        <h3 class="text-purple-300 font-bold mb-2">盲文输出</h3>
        <div class="text-4xl tracking-wider text-purple-300 h-16">{{ store.brailleUnicode }}</div>
        <div class="flex flex-wrap gap-2 mt-3">
          <BrailleCell v-for="(dots, i) in store.brailleOutput" :key="i" :dots="dots" :size="40" />
        </div>
      </div>
    </div>

    <!-- Learn -->
    <div v-if="activeTab === 'learn'" class="grid grid-cols-2 gap-4">
      <div class="bg-gray-900 rounded-xl p-4 flex flex-col items-center gap-4">
        <h3 class="text-purple-300 font-bold">猜盲文</h3>
        <div v-if="!store.quizChar">
          <button @click="store.generateQuiz()" class="bg-purple-500 px-6 py-3 rounded-lg text-lg hover:bg-purple-400">
            开始训练
          </button>
        </div>
        <div v-else class="flex flex-col items-center gap-3">
          <div class="text-7xl font-bold text-purple-400">{{ store.quizChar }}</div>

          <template v-if="!store.showResult">
            <div class="text-sm text-gray-400">点击下方 6 点阵选择对应盲文</div>
            <div class="grid grid-cols-2 gap-2 p-4 bg-gray-800 rounded-xl">
              <button v-for="d in 6" :key="d" @click="store.toggleDot(d)"
                class="w-14 h-14 rounded-full border-2 transition-all"
                :class="store.selectedDots.includes(d) ? 'bg-purple-500 border-purple-400 scale-110' : 'bg-gray-700 border-gray-600 hover:border-purple-400'">
                <span class="text-xs">{{ d }}</span>
              </button>
            </div>
            <button @click="store.checkQuizAnswer()" class="bg-purple-500 px-6 py-2 rounded hover:bg-purple-400">确认</button>
          </template>

          <template v-else>
            <div class="w-full bg-gray-800 rounded-xl p-4 flex flex-col items-center gap-3">
              <div v-if="store.missedDots.length === 0 && store.extraDots.length === 0"
                class="text-green-400 text-lg font-bold">
                ✓ 回答正确！
              </div>
              <div v-else class="flex flex-col items-center gap-2">
                <div class="text-red-400 text-lg font-bold">✗ 回答错误</div>
                <div class="flex gap-4 text-sm">
                  <div v-if="store.missedDots.length" class="text-yellow-400">
                    遗漏点位：{{ store.missedDots.join(', ') }}
                  </div>
                  <div v-if="store.extraDots.length" class="text-orange-400">
                    多选点位：{{ store.extraDots.join(', ') }}
                  </div>
                </div>
              </div>

              <div class="w-full border-t border-gray-700 pt-3 mt-1">
                <div class="text-sm text-gray-400 mb-2 text-center">正确答案</div>
                <div class="flex items-center justify-center gap-3">
                  <span class="text-2xl font-bold text-purple-400">{{ store.quizChar }}</span>
                  <span class="text-gray-500">→</span>
                  <BrailleCell :dots="store.correctDots" :size="50" />
                  <span class="text-sm text-gray-400">[{{ store.correctDots.join(', ') }}]</span>
                </div>
              </div>

              <div class="w-full border-t border-gray-700 pt-3 mt-1">
                <div class="text-sm text-gray-400 mb-2 text-center">你的选择 vs 正确答案</div>
                <div class="flex items-center justify-center gap-6">
                  <div class="flex flex-col items-center gap-1">
                    <span class="text-xs text-gray-500">你的选择</span>
                    <div class="grid grid-cols-2 gap-1 p-2 bg-gray-900 rounded-lg">
                      <div v-for="d in 6" :key="'user-'+d"
                        class="w-10 h-10 rounded-full border-2 flex items-center justify-center text-xs"
                        :class="store.selectedDots.includes(d)
                          ? (store.correctDots.includes(d) ? 'bg-green-500 border-green-400' : 'bg-orange-500 border-orange-400')
                          : (store.correctDots.includes(d) ? 'bg-yellow-600 border-yellow-500' : 'bg-gray-700 border-gray-600')">
                        {{ d }}
                      </div>
                    </div>
                  </div>
                  <div class="flex flex-col items-center gap-1">
                    <span class="text-xs text-gray-500">正确答案</span>
                    <div class="grid grid-cols-2 gap-1 p-2 bg-gray-900 rounded-lg">
                      <div v-for="d in 6" :key="'correct-'+d"
                        class="w-10 h-10 rounded-full border-2 flex items-center justify-center text-xs"
                        :class="store.correctDots.includes(d) ? 'bg-purple-500 border-purple-400' : 'bg-gray-700 border-gray-600'">
                        {{ d }}
                      </div>
                    </div>
                  </div>
                </div>
                <div class="flex justify-center gap-4 mt-2 text-xs">
                  <span class="flex items-center gap-1"><span class="w-3 h-3 rounded-full bg-green-500 inline-block"></span> 正确</span>
                  <span class="flex items-center gap-1"><span class="w-3 h-3 rounded-full bg-yellow-600 inline-block"></span> 遗漏</span>
                  <span class="flex items-center gap-1"><span class="w-3 h-3 rounded-full bg-orange-500 inline-block"></span> 多选</span>
                </div>
              </div>
            </div>

            <button @click="store.nextQuiz()" class="bg-purple-500 px-6 py-2 rounded hover:bg-purple-400">下一题</button>
          </template>
        </div>
      </div>
      <div class="bg-gray-900 rounded-xl p-4">
        <div class="flex justify-between mb-2">
          <h3 class="text-purple-300 font-bold">统计</h3>
          <button @click="store.resetScore()" class="text-red-400 text-xs hover:underline">重置</button>
        </div>
        <div class="grid grid-cols-3 gap-2 text-center mb-3">
          <div class="bg-gray-800 rounded p-2">
            <div class="text-2xl font-bold text-green-400">{{ store.score.correct }}</div>
            <div class="text-xs text-gray-400">正确</div>
          </div>
          <div class="bg-gray-800 rounded p-2">
            <div class="text-2xl font-bold text-red-400">{{ store.score.total - store.score.correct }}</div>
            <div class="text-xs text-gray-400">错误</div>
          </div>
          <div class="bg-gray-800 rounded p-2">
            <div class="text-2xl font-bold text-purple-400">{{ store.score.total ? Math.round(store.score.correct / store.score.total * 100) : 0 }}%</div>
            <div class="text-xs text-gray-400">正确率</div>
          </div>
        </div>
        <div class="space-y-1 max-h-48 overflow-y-auto">
          <div v-for="(h, i) in store.history.slice(0, 20)" :key="i"
            class="flex justify-between bg-gray-800 rounded p-2 text-sm"
            :class="h.correct ? 'border-l-4 border-green-500' : 'border-l-4 border-red-500'">
            <span>{{ h.input }}</span><span>{{ h.correct ? '✓' : '✗' }}</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Reference -->
    <div v-if="activeTab === 'ref'" class="bg-gray-900 rounded-xl p-4">
      <h3 class="text-purple-300 font-bold mb-3">盲文速查表</h3>
      <div class="grid grid-cols-6 md:grid-cols-9 gap-3">
        <div v-for="(dots, char) in brailleMap" :key="char" class="flex flex-col items-center">
          <div class="text-xl font-bold text-purple-400">{{ char }}</div>
          <BrailleCell :dots="dots" :size="30" />
          <div class="text-xs text-gray-500">{{ dots.join(',') }}</div>
        </div>
      </div>
    </div>

    <button @click="doExport" class="bg-green-700 px-4 py-2 rounded self-start hover:bg-green-600 text-sm">
      导出翻译文本
    </button>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useBrailleStore } from './store/braille'
import { BRAILLE_MAP } from './utils/braille'
import BrailleCell from './components/BrailleCell.vue'

const store = useBrailleStore()
const brailleMap = BRAILLE_MAP
const tabs = [
  { id: 'translate', label: '翻译模式' },
  { id: 'learn', label: '训练模式' },
  { id: 'ref', label: '速查表' },
]
const activeTab = ref('translate')

function doExport() {
  const text = store.exportPDF()
  const blob = new Blob([text], { type: 'text/plain' })
  const a = document.createElement('a')
  a.href = URL.createObjectURL(blob)
  a.download = 'braille-output.txt'
  a.click()
}
</script>
