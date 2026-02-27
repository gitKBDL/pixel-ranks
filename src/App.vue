<template>
  <div class="app">
    <section class="editor-shell">
      <header class="editor-header">
        <div class="editor-heading">
          <p class="editor-kicker">Пиксельные ранги</p>
          <h1 class="editor-title">Редактор бейджей</h1>
          <p class="editor-subtitle">Создавайте, настраивайте и экспортируйте пиксельные ранги.</p>
        </div>

        <div class="header-actions">
          <button @click="copyImage" class="dark-button ghost-button" :disabled="!imageSrc">
            Копировать PNG
          </button>
          <button @click="downloadImage" class="dark-button primary-button" :disabled="!imageSrc">
            Скачать PNG
          </button>
        </div>
      </header>

      <section class="preview-section panel-block">
        <div class="preview-topline">
          <label class="field-label" for="rank-text-input">Текст ранга</label>
          <p class="preview-meta">{{ text.length }} симв. · {{ width }}x{{ height + 1 }} px</p>
        </div>

        <div class="text-input-row">
          <input
            id="rank-text-input"
            v-model="text"
            placeholder="Введите текст ранга..."
            class="input dark-input"
          />
          <button @click="text = ''" class="dark-button subtle-button clear-button" :disabled="!text">
            Очистить
          </button>
        </div>

        <canvas ref="canvas" class="preview-canvas"></canvas>
        <div class="preview-frame">
          <img
            :src="imageSrc"
            alt="Предпросмотр"
            class="preview-img"
            :style="{ backgroundColor: bgColor, borderColor: borderColor }"
          />
        </div>

        <p v-if="errorText" class="error-text">{{ errorText }}</p>
        <p v-if="noticeText" :class="['notice-text', `notice-${noticeType}`]">{{ noticeText }}</p>
      </section>

      <section class="settings-shell">
        <div class="settings-block panel-block">
          <h2 class="settings-title">Цвета слоёв</h2>
          <div class="color-pickers">
            <div class="color-row">
              <span class="row-title">Фон</span>
              <div class="row-controls">
                <input type="color" v-model="bgColor" class="color-picker" />
                <input
                  v-if="useBgGradient"
                  type="color"
                  v-model="bgGradientColor"
                  class="color-picker"
                />
                <label class="inline-toggle">
                  <input type="checkbox" v-model="useBgGradient" />
                  Градиент
                </label>
              </div>
            </div>

            <div class="color-row">
              <span class="row-title">Рамка</span>
              <div class="row-controls">
                <input type="color" v-model="borderColor" class="color-picker" />
                <input
                  v-if="useBorderGradient"
                  type="color"
                  v-model="borderGradientColor"
                  class="color-picker"
                />
                <label class="inline-toggle">
                  <input type="checkbox" v-model="showBorder" />
                  Включено
                </label>
                <label class="inline-toggle">
                  <input type="checkbox" v-model="useBorderGradient" />
                  Градиент
                </label>
              </div>
            </div>

            <div class="color-row">
              <span class="row-title">Тень</span>
              <div class="row-controls">
                <input type="color" v-model="shadowColor" class="color-picker" />
                <input
                  v-if="useShadowGradient"
                  type="color"
                  v-model="shadowGradientColor"
                  class="color-picker"
                />
                <label class="inline-toggle">
                  <input type="checkbox" v-model="showShadow" />
                  Включено
                </label>
                <label class="inline-toggle">
                  <input type="checkbox" v-model="useShadowGradient" />
                  Градиент
                </label>
              </div>
            </div>

            <div class="color-row">
              <span class="row-title">Текст</span>
              <div class="row-controls">
                <input type="color" v-model="textColor" class="color-picker" />
                <input
                  v-if="useTextGradient"
                  type="color"
                  v-model="textGradientColor"
                  class="color-picker"
                />
                <label class="inline-toggle">
                  <input type="checkbox" v-model="useTextGradient" />
                  Градиент
                </label>
              </div>
            </div>
          </div>
        </div>

        <div class="settings-block compact-block panel-block">
          <h2 class="settings-title">Общие настройки</h2>
          <div class="stacked-fields">
            <label class="field-label">Направление градиента</label>
            <select v-model="gradientDirection" class="dark-select">
              <option value="horizontal">Горизонтальное</option>
              <option value="vertical">Вертикальное</option>
              <option value="diagonal">Диагональное</option>
            </select>

            <label class="field-label">Пресет</label>
            <select v-model="selectedPreset" @change="applyPreset" class="dark-select">
              <option disabled value="">Выберите пресет</option>
              <option v-for="(_, name) in presets" :key="name" :value="name">
                {{ presetLabels[name] ?? name }}
              </option>
            </select>
          </div>

          <div class="quick-actions">
            <button @click="resetAppearance" class="dark-button subtle-button">Сбросить цвета</button>
            <button @click="fillProjectNameFromText" class="dark-button subtle-button">
              Имя из текста
            </button>
          </div>
        </div>
      </section>
    </section>

    <aside class="projects-panel">
      <div class="projects-header panel-block">
        <div>
          <h2 class="projects-title">Проекты</h2>
          <p class="projects-subtitle">Хранятся локально в браузере</p>
        </div>
        <p class="projects-count">Всего: {{ savedProjects.length }}</p>
      </div>

      <section class="projects-controls panel-block">
        <input
          v-model="projectName"
          placeholder="Название проекта (необязательно)..."
          class="input dark-input project-name-input"
        />

        <input
          v-model="projectSearch"
          placeholder="Поиск проектов..."
          class="input dark-input project-search-input"
        />

        <div class="projects-toolbar">
          <button @click="saveProject" class="dark-button">
            {{ openedProjectId ? 'Обновить проект' : 'Сохранить проект' }}
          </button>
          <button v-if="openedProjectId" @click="saveAsNewProject" class="dark-button">
            Сохранить как новый
          </button>
          <button @click="startNewProject" class="dark-button">Новый черновик</button>
        </div>

        <p v-if="projectError" class="error-text">{{ projectError }}</p>
      </section>

      <section class="projects-list-shell panel-block">
        <div class="projects-list">
          <p v-if="!savedProjects.length" class="projects-empty">Пока нет сохранённых проектов.</p>
          <p v-else-if="!filteredProjects.length" class="projects-empty">
            Нет проектов по вашему запросу.
          </p>

          <article
            v-for="project in filteredProjects"
            :key="project.id"
            class="project-card"
            :class="{ 'project-card-opened': openedProjectId === project.id }"
          >
            <img
              v-if="project.preview"
              :src="project.preview"
              :alt="`Предпросмотр ${project.name}`"
              class="project-preview"
            />
            <div v-else class="project-preview project-preview-empty">Нет превью</div>

            <div class="project-info">
              <p class="project-name">{{ project.name }}</p>
              <p class="project-meta">{{ formatProjectDate(project.updatedAt) }}</p>
              <p class="project-meta">Текст: {{ project.text }}</p>
              <p v-if="openedProjectId === project.id" class="project-opened">Открыт сейчас</p>
            </div>

            <div class="project-actions">
              <button @click="openProject(project.id)" class="dark-button">Открыть</button>
              <button @click="deleteProject(project.id)" class="dark-button danger-button">Удалить</button>
            </div>
          </article>
        </div>
      </section>
    </aside>
  </div>
</template>

<script setup>
import { computed, ref, watch, onMounted, onBeforeUnmount } from 'vue'

const PROJECTS_STORAGE_KEY = 'pixel-ranks.projects.v1'
const OPENED_PROJECT_STORAGE_KEY = 'pixel-ranks.opened-project.v1'

const DEFAULT_APPEARANCE = Object.freeze({
  bgColor: '#282828',
  useBgGradient: false,
  bgGradientColor: '#3f3f3f',
  borderColor: '#a0a0a0',
  showBorder: true,
  useBorderGradient: false,
  borderGradientColor: '#ffffff',
  shadowColor: '#505050',
  showShadow: true,
  useShadowGradient: false,
  shadowGradientColor: '#b7b7b7',
  textColor: '#ffffff',
  useTextGradient: false,
  textGradientColor: '#7cd5ff',
  gradientDirection: 'horizontal'
})

const text = ref('Ранг')
const canvas = ref(null)
const tileSize = 8
const padding = 1
const spacing = 1
const height = tileSize
const width = ref(0)
const imageSrc = ref('')

const bgColor = ref(DEFAULT_APPEARANCE.bgColor)
const borderColor = ref(DEFAULT_APPEARANCE.borderColor)
const shadowColor = ref(DEFAULT_APPEARANCE.shadowColor)
const textColor = ref(DEFAULT_APPEARANCE.textColor)
const selectedPreset = ref('')
const showBorder = ref(DEFAULT_APPEARANCE.showBorder)
const showShadow = ref(DEFAULT_APPEARANCE.showShadow)
const useBgGradient = ref(DEFAULT_APPEARANCE.useBgGradient)
const bgGradientColor = ref(DEFAULT_APPEARANCE.bgGradientColor)
const useBorderGradient = ref(DEFAULT_APPEARANCE.useBorderGradient)
const borderGradientColor = ref(DEFAULT_APPEARANCE.borderGradientColor)
const useShadowGradient = ref(DEFAULT_APPEARANCE.useShadowGradient)
const shadowGradientColor = ref(DEFAULT_APPEARANCE.shadowGradientColor)
const useTextGradient = ref(DEFAULT_APPEARANCE.useTextGradient)
const textGradientColor = ref(DEFAULT_APPEARANCE.textGradientColor)
const gradientDirection = ref(DEFAULT_APPEARANCE.gradientDirection)

const errorText = ref('')
const noticeText = ref('')
const noticeType = ref('info')
const projectName = ref('')
const projectSearch = ref('')
const projectError = ref('')
const savedProjects = ref([])
const openedProjectId = ref('')

let noticeTimerId = null

const saturation = 0.08
const presetGradientSaturation = 0.14
const presets = {
  Classic: { bg: '#282828', border: '#a0a0a0' },
  Emerald: { bg: '#003e2f', border: '#00ffba' },
  Gold: { bg: '#3b2c00', border: '#ffcc00' },
  Nether: { bg: '#2b0f0f', border: '#ff3b3b' },
  Ice: { bg: '#0f2b3b', border: '#3bafff' },
  Diamond: { bg: '#0f3b3b', border: '#3bffff' },
  Ruby: { bg: '#3b0f0f', border: '#ff3b6b' },
  Amethyst: { bg: '#2b0f3b', border: '#a03bff' },
  Obsidian: { bg: '#0f0f2b', border: '#3b3bff' },
  Sandstone: { bg: '#3b2b0f', border: '#ffcc66' },
  Lapis: { bg: '#0f0f3b', border: '#3b6bff' },
  Ender: { bg: '#1a0f2b', border: '#7f3bff' },
  Prismarine: { bg: '#0f3b2b', border: '#3bffcc' },
  Copper: { bg: '#3b1f0f', border: '#ff9966' },
  Glowstone: { bg: '#3b3b0f', border: '#ffff66' },
  Crimson: { bg: '#3b0f1f', border: '#ff6699' },
  Warped: { bg: '#0f3b1f', border: '#66ff99' }
}

const presetLabels = {
  Classic: 'Классика',
  Emerald: 'Изумруд',
  Gold: 'Золото',
  Nether: 'Незер',
  Ice: 'Лёд',
  Diamond: 'Алмаз',
  Ruby: 'Рубин',
  Amethyst: 'Аметист',
  Obsidian: 'Обсидиан',
  Sandstone: 'Песчаник',
  Lapis: 'Лазурит',
  Ender: 'Эндер',
  Prismarine: 'Призмарин',
  Copper: 'Медь',
  Glowstone: 'Светокамень',
  Crimson: 'Багрянец',
  Warped: 'Искажённый лес'
}

for (const name in presets) {
  const preset = presets[name]
  preset.shadow = adjustHSL(preset.bg, saturation)
  preset.bgGradient = adjustHSL(preset.bg, presetGradientSaturation)
  preset.borderGradient = adjustHSL(preset.border, presetGradientSaturation)
  preset.shadowGradient = adjustHSL(preset.shadow, presetGradientSaturation)
  preset.text = '#ffffff'
  preset.textGradient = adjustHSL(preset.border, 0.06)
}

const filteredProjects = computed(() => {
  const query = projectSearch.value.trim().toLowerCase()
  if (!query) {
    return savedProjects.value
  }

  return savedProjects.value.filter(project => {
    const haystack = `${project.name} ${project.text}`.toLowerCase()
    return haystack.includes(query)
  })
})

function setNotice(message, type = 'info') {
  noticeText.value = message
  noticeType.value = type

  if (noticeTimerId) {
    window.clearTimeout(noticeTimerId)
  }

  noticeTimerId = window.setTimeout(() => {
    noticeText.value = ''
  }, 2300)
}

function setAppearanceFromState(state) {
  bgColor.value = state.bgColor
  useBgGradient.value = state.useBgGradient
  bgGradientColor.value = state.bgGradientColor
  borderColor.value = state.borderColor
  showBorder.value = state.showBorder
  useBorderGradient.value = state.useBorderGradient
  borderGradientColor.value = state.borderGradientColor
  shadowColor.value = state.shadowColor
  showShadow.value = state.showShadow
  useShadowGradient.value = state.useShadowGradient
  shadowGradientColor.value = state.shadowGradientColor
  textColor.value = state.textColor
  useTextGradient.value = state.useTextGradient
  textGradientColor.value = state.textGradientColor
  gradientDirection.value = state.gradientDirection
}

function resetAppearance() {
  setAppearanceFromState(DEFAULT_APPEARANCE)
  selectedPreset.value = ''
  setNotice('Оформление сброшено', 'info')
}

function fillProjectNameFromText() {
  const fromText = text.value.trim().replace(/\s+/g, ' ').slice(0, 28)
  projectName.value = fromText || projectName.value || 'Проект без названия'
}

function buildFallbackProjectName() {
  const fromText = text.value.trim().replace(/\s+/g, ' ').slice(0, 28)
  if (fromText) {
    return fromText
  }

  return `Ранг ${new Date().toLocaleString('ru-RU', {
    month: 'short',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  })}`
}

function adjustHSL(colorHex, lightnessAdjustment) {
  const r = parseInt(colorHex.slice(1, 3), 16) / 255
  const g = parseInt(colorHex.slice(3, 5), 16) / 255
  const b = parseInt(colorHex.slice(5, 7), 16) / 255

  const max = Math.max(r, g, b)
  const min = Math.min(r, g, b)
  const l = (max + min) / 2

  const s = max === min ? 0 : l < 0.5 ? (max - min) / (max + min) : (max - min) / (2 - max - min)

  const h = (() => {
    if (max === min) return 0
    if (max === r) return ((g - b) / (max - min) + (g < b ? 6 : 0)) / 6
    if (max === g) return ((b - r) / (max - min) + 2) / 6
    return ((r - g) / (max - min) + 4) / 6
  })()

  const adjustedL = Math.min(Math.max(l + lightnessAdjustment, 0), 1)

  const q = adjustedL < 0.5 ? adjustedL * (1 + s) : adjustedL + s - adjustedL * s
  const p = 2 * adjustedL - q

  const toRGB = t => {
    if (t < 0) t += 1
    if (t > 1) t -= 1
    if (t < 1 / 6) return p + (q - p) * 6 * t
    if (t < 1 / 2) return q
    if (t < 2 / 3) return p + (q - p) * (2 / 3 - t) * 6
    return p
  }

  const newR = Math.round(toRGB(h + 1 / 3) * 255)
  const newG = Math.round(toRGB(h) * 255)
  const newB = Math.round(toRGB(h - 1 / 3) * 255)

  return `#${newR.toString(16).padStart(2, '0')}${newG.toString(16).padStart(2, '0')}${newB.toString(16).padStart(2, '0')}`
}

function createGradient(ctx, width, height, startColor, endColor) {
  let x0 = 0
  let y0 = 0
  let x1 = width
  let y1 = 0

  if (gradientDirection.value === 'vertical') {
    x1 = 0
    y1 = height
  } else if (gradientDirection.value === 'diagonal') {
    x1 = width
    y1 = height
  }

  const gradient = ctx.createLinearGradient(x0, y0, x1, y1)
  gradient.addColorStop(0, startColor)
  gradient.addColorStop(1, endColor)

  return gradient
}

const applyPreset = () => {
  const preset = presets[selectedPreset.value]
  if (preset) {
    bgColor.value = preset.bg
    useBgGradient.value = true
    bgGradientColor.value = preset.bgGradient

    borderColor.value = preset.border
    showBorder.value = true
    useBorderGradient.value = true
    borderGradientColor.value = preset.borderGradient

    shadowColor.value = preset.shadow
    showShadow.value = true
    useShadowGradient.value = true
    shadowGradientColor.value = preset.shadowGradient

    textColor.value = preset.text
    useTextGradient.value = true
    textGradientColor.value = preset.textGradient
  }
}

const fontImage = new Image()
const fontAssetPath = `${import.meta.env.BASE_URL.replace(/\/?$/, '/')}ascii.png`
fontImage.src = fontAssetPath

let charWidths = {}

const charToIndex = char => {
  const code = char.charCodeAt(0)
  return code >= 32 && code <= 127 ? code - 32 : 0
}

const analyzeCharWidths = () => {
  const offCanvas = document.createElement('canvas')
  offCanvas.width = 128
  offCanvas.height = 128
  const ctx = offCanvas.getContext('2d')
  ctx.drawImage(fontImage, 0, 0)

  for (let i = 0; i < 96; i++) {
    const sx = (i % 16) * tileSize
    const sy = Math.floor(i / 16) * tileSize
    const imageData = ctx.getImageData(sx, sy, tileSize, tileSize)
    const data = imageData.data

    let minX = tileSize
    let maxX = 0

    for (let y = 0; y < tileSize; y++) {
      for (let x = 0; x < tileSize; x++) {
        const idx = (y * tileSize + x) * 4
        const alpha = data[idx + 3]
        if (alpha > 0) {
          if (x < minX) minX = x
          if (x > maxX) maxX = x
        }
      }
    }

    const measuredWidth = maxX >= minX ? maxX - minX + 1 : 0
    charWidths[i] = { width: measuredWidth, offsetX: minX }
  }
}

function transliterateForFont(inputText) {
  const map = {
    а: 'a',
    б: 'b',
    в: 'v',
    г: 'g',
    д: 'd',
    е: 'e',
    ё: 'yo',
    ж: 'zh',
    з: 'z',
    и: 'i',
    й: 'y',
    к: 'k',
    л: 'l',
    м: 'm',
    н: 'n',
    о: 'o',
    п: 'p',
    р: 'r',
    с: 's',
    т: 't',
    у: 'u',
    ф: 'f',
    х: 'h',
    ц: 'c',
    ч: 'ch',
    ш: 'sh',
    щ: 'sch',
    ъ: '',
    ы: 'y',
    ь: '',
    э: 'e',
    ю: 'yu',
    я: 'ya'
  }

  return inputText
    .toLowerCase()
    .split('')
    .map(char => map[char] ?? char)
    .join('')
    .replace(/[^\x20-\x7e]/g, ' ')
}

const draw = () => {
  if (!canvas.value || !fontImage.complete) return

  const ctx = canvas.value.getContext('2d')
  if (!ctx) return
  ctx.imageSmoothingEnabled = false

  const chars = transliterateForFont(text.value).split('')
  let totalWidth = padding + 1

  for (const char of chars) {
    const i = charToIndex(char)
    const measuredWidth = charWidths[i]?.width ?? tileSize
    totalWidth += measuredWidth + spacing
  }

  const borderExtension = showBorder.value && showShadow.value ? 1 : 0
  const finalWidth = totalWidth - spacing + 2 + borderExtension
  const finalHeight = height + 1

  canvas.value.width = finalWidth
  canvas.value.height = finalHeight
  width.value = finalWidth

  ctx.fillStyle = useBgGradient.value
    ? createGradient(ctx, finalWidth, finalHeight, bgColor.value, bgGradientColor.value)
    : bgColor.value
  ctx.fillRect(0, 0, finalWidth, finalHeight)

  if (showBorder.value) {
    ctx.fillStyle = useBorderGradient.value
      ? createGradient(ctx, finalWidth, finalHeight, borderColor.value, borderGradientColor.value)
      : borderColor.value
    ctx.fillRect(0, 0, finalWidth, 1)
    ctx.fillRect(0, finalHeight - 1, finalWidth, 1)
    ctx.fillRect(0, 0, 1, finalHeight)
    ctx.fillRect(finalWidth - 1, 0, 1, finalHeight)
  }

  const glyphs = []
  let cursor = padding + 1
  for (const char of chars) {
    const i = charToIndex(char)
    const tileX = (i % 16) * tileSize
    const tileY = Math.floor(i / 16) * tileSize
    const info = charWidths[i] ?? { width: tileSize, offsetX: 0 }

    glyphs.push({
      sourceX: tileX + info.offsetX,
      sourceY: tileY,
      width: info.width,
      targetX: cursor
    })

    cursor += info.width + spacing
  }

  if (showShadow.value) {
    const shadowLayer = document.createElement('canvas')
    shadowLayer.width = finalWidth
    shadowLayer.height = finalHeight
    const shadowCtx = shadowLayer.getContext('2d')

    if (shadowCtx) {
      shadowCtx.imageSmoothingEnabled = false
      for (const glyph of glyphs) {
        shadowCtx.drawImage(
          fontImage,
          glyph.sourceX,
          glyph.sourceY,
          glyph.width,
          tileSize,
          glyph.targetX + 1,
          0,
          glyph.width,
          tileSize
        )
      }

      shadowCtx.globalCompositeOperation = 'source-in'
      shadowCtx.fillStyle = useShadowGradient.value
        ? createGradient(shadowCtx, finalWidth, finalHeight, shadowColor.value, shadowGradientColor.value)
        : shadowColor.value
      shadowCtx.fillRect(0, 0, finalWidth, finalHeight)
      shadowCtx.globalCompositeOperation = 'source-over'

      ctx.drawImage(shadowLayer, 0, 0)
    }
  }

  const needsTextMask = useTextGradient.value || textColor.value.toLowerCase() !== '#ffffff'
  if (needsTextMask) {
    const textLayer = document.createElement('canvas')
    textLayer.width = finalWidth
    textLayer.height = finalHeight
    const textCtx = textLayer.getContext('2d')

    if (textCtx) {
      textCtx.imageSmoothingEnabled = false
      for (const glyph of glyphs) {
        textCtx.drawImage(
          fontImage,
          glyph.sourceX,
          glyph.sourceY,
          glyph.width,
          tileSize,
          glyph.targetX,
          0,
          glyph.width,
          tileSize
        )
      }

      textCtx.globalCompositeOperation = 'source-in'
      textCtx.fillStyle = useTextGradient.value
        ? createGradient(textCtx, finalWidth, finalHeight, textColor.value, textGradientColor.value)
        : textColor.value
      textCtx.fillRect(0, 0, finalWidth, finalHeight)
      textCtx.globalCompositeOperation = 'source-over'

      ctx.drawImage(textLayer, 0, 0)
    }
  } else {
    for (const glyph of glyphs) {
      ctx.drawImage(
        fontImage,
        glyph.sourceX,
        glyph.sourceY,
        glyph.width,
        tileSize,
        glyph.targetX,
        0,
        glyph.width,
        tileSize
      )
    }
  }

  imageSrc.value = canvas.value.toDataURL()
}

function downloadImage() {
  if (!imageSrc.value) {
    return
  }

  const link = document.createElement('a')
  link.href = imageSrc.value
  link.download = 'rang.png'
  link.click()
  setNotice('PNG скачан', 'success')
}

async function createPngBlobFromCanvas(canvasElement) {
  const directBlob = await new Promise(resolve => {
    if (typeof canvasElement.toBlob !== 'function') {
      resolve(null)
      return
    }

    canvasElement.toBlob(result => {
      resolve(result ?? null)
    }, 'image/png')
  })

  if (directBlob) {
    return directBlob
  }

  if (!imageSrc.value) {
    return null
  }

  try {
    const response = await fetch(imageSrc.value)
    return await response.blob()
  } catch {
    return null
  }
}

function createClipboardItemFromCanvas(canvasElement) {
  if (typeof ClipboardItem === 'undefined' || typeof canvasElement.toBlob !== 'function') {
    return null
  }

  const pngPromise = new Promise((resolve, reject) => {
    canvasElement.toBlob(result => {
      if (result) {
        resolve(result)
        return
      }
      reject(new Error('Не удалось подготовить изображение'))
    }, 'image/png')
  })

  return new ClipboardItem({ 'image/png': pngPromise })
}

function getClipboardErrorMessage(error) {
  if (error && typeof error === 'object' && 'name' in error) {
    if (error.name === 'NotAllowedError') {
      return 'Браузер отклонил доступ к буферу (часто без диалога). Разрешите буфер для сайта.'
    }
    if (error.name === 'NotSupportedError') {
      return 'Копирование PNG не поддерживается в этом браузере.'
    }
  }

  return 'Не удалось скопировать PNG. Можно скачать файл кнопкой рядом.'
}

async function copyImage() {
  if (!canvas.value || !imageSrc.value) {
    return
  }

  if (!window.isSecureContext) {
    setNotice('Копирование PNG работает только на HTTPS или localhost', 'error')
    return
  }

  try {
    const hasNativeImageCopySupport = Boolean(
      navigator.clipboard &&
        typeof navigator.clipboard.write === 'function' &&
        typeof ClipboardItem !== 'undefined'
    )

    if (hasNativeImageCopySupport) {
      try {
        const directClipboardItem = createClipboardItemFromCanvas(canvas.value)
        if (directClipboardItem) {
          await navigator.clipboard.write([directClipboardItem])
          setNotice('PNG скопирован в буфер обмена', 'success')
          return
        }

        const fallbackBlob = await createPngBlobFromCanvas(canvas.value)
        if (fallbackBlob) {
          const mimeType = fallbackBlob.type || 'image/png'
          await navigator.clipboard.write([new ClipboardItem({ [mimeType]: fallbackBlob })])
          setNotice('PNG скопирован в буфер обмена', 'success')
          return
        }
      } catch (nativeCopyError) {
        if (!navigator.clipboard || typeof navigator.clipboard.writeText !== 'function') {
          setNotice(getClipboardErrorMessage(nativeCopyError), 'error')
          return
        }
      }
    }

    const blob = await createPngBlobFromCanvas(canvas.value)
    if (!blob && (!navigator.clipboard || typeof navigator.clipboard.writeText !== 'function')) {
      setNotice('Не удалось подготовить PNG для копирования', 'error')
      return
    }

    if (navigator.clipboard && typeof navigator.clipboard.writeText === 'function') {
      await navigator.clipboard.writeText(imageSrc.value)
      setNotice('PNG как изображение недоступен. Скопирована data-ссылка.', 'info')
      return
    }

    setNotice('Копирование в буфер не поддерживается в этом браузере', 'error')
  } catch (error) {
    setNotice(getClipboardErrorMessage(error), 'error')
  }
}

function createProjectId() {
  if (typeof crypto !== 'undefined' && crypto.randomUUID) {
    return crypto.randomUUID()
  }
  return `${Date.now()}-${Math.random().toString(16).slice(2)}`
}

function normalizeProject(rawProject) {
  const source = rawProject && typeof rawProject === 'object' ? rawProject : {}
  const fallbackDate = new Date().toISOString()
  const parsedDate = Date.parse(source.updatedAt ?? '')

  return {
    id: String(source.id ?? createProjectId()),
    name: String(source.name ?? 'Проект без названия'),
    text: String(source.text ?? 'Ранг'),
    bgColor: String(source.bgColor ?? DEFAULT_APPEARANCE.bgColor),
    useBgGradient: source.useBgGradient === true,
    bgGradientColor: String(source.bgGradientColor ?? DEFAULT_APPEARANCE.bgGradientColor),
    borderColor: String(source.borderColor ?? DEFAULT_APPEARANCE.borderColor),
    useBorderGradient: source.useBorderGradient === true,
    borderGradientColor: String(source.borderGradientColor ?? DEFAULT_APPEARANCE.borderGradientColor),
    shadowColor: String(source.shadowColor ?? DEFAULT_APPEARANCE.shadowColor),
    useShadowGradient: source.useShadowGradient === true,
    shadowGradientColor: String(source.shadowGradientColor ?? DEFAULT_APPEARANCE.shadowGradientColor),
    textColor: String(source.textColor ?? DEFAULT_APPEARANCE.textColor),
    useTextGradient: source.useTextGradient === true,
    textGradientColor: String(source.textGradientColor ?? DEFAULT_APPEARANCE.textGradientColor),
    gradientDirection: ['horizontal', 'vertical', 'diagonal'].includes(source.gradientDirection)
      ? source.gradientDirection
      : DEFAULT_APPEARANCE.gradientDirection,
    showBorder: source.showBorder !== false,
    showShadow: source.showShadow !== false,
    selectedPreset: String(source.selectedPreset ?? ''),
    preview: String(source.preview ?? ''),
    updatedAt: Number.isNaN(parsedDate) ? fallbackDate : new Date(parsedDate).toISOString()
  }
}

function sortProjects(projects) {
  return [...projects].sort((a, b) => Date.parse(b.updatedAt) - Date.parse(a.updatedAt))
}

function persistProjects() {
  localStorage.setItem(PROJECTS_STORAGE_KEY, JSON.stringify(savedProjects.value))
}

function persistOpenedProject() {
  if (openedProjectId.value) {
    localStorage.setItem(OPENED_PROJECT_STORAGE_KEY, openedProjectId.value)
    return
  }
  localStorage.removeItem(OPENED_PROJECT_STORAGE_KEY)
}

function saveProject() {
  const isUpdating = Boolean(openedProjectId.value)
  let normalizedName = projectName.value.trim()

  if (!normalizedName) {
    normalizedName = buildFallbackProjectName()
    projectName.value = normalizedName
  }

  if (fontImage.complete) {
    draw()
  }

  const projectId = openedProjectId.value || createProjectId()
  const updatedProject = {
    id: projectId,
    name: normalizedName,
    text: text.value,
    bgColor: bgColor.value,
    useBgGradient: useBgGradient.value,
    bgGradientColor: bgGradientColor.value,
    borderColor: borderColor.value,
    useBorderGradient: useBorderGradient.value,
    borderGradientColor: borderGradientColor.value,
    shadowColor: shadowColor.value,
    useShadowGradient: useShadowGradient.value,
    shadowGradientColor: shadowGradientColor.value,
    textColor: textColor.value,
    useTextGradient: useTextGradient.value,
    textGradientColor: textGradientColor.value,
    gradientDirection: gradientDirection.value,
    showBorder: showBorder.value,
    showShadow: showShadow.value,
    selectedPreset: selectedPreset.value,
    preview: imageSrc.value,
    updatedAt: new Date().toISOString()
  }

  const existingIndex = savedProjects.value.findIndex(project => project.id === projectId)
  if (existingIndex >= 0) {
    savedProjects.value[existingIndex] = updatedProject
  } else {
    savedProjects.value.push(updatedProject)
  }

  savedProjects.value = sortProjects(savedProjects.value)
  openedProjectId.value = projectId
  projectName.value = normalizedName
  projectError.value = ''
  persistProjects()
  persistOpenedProject()
  setNotice(isUpdating ? 'Проект обновлён' : 'Проект сохранён', 'success')
}

function saveAsNewProject() {
  openedProjectId.value = ''
  persistOpenedProject()
  saveProject()
}

function openProject(projectId, silent = false) {
  const project = savedProjects.value.find(item => item.id === projectId)
  if (!project) {
    if (!silent) {
      projectError.value = 'Проект не найден'
    }
    return
  }

  openedProjectId.value = project.id
  projectName.value = project.name
  text.value = project.text
  bgColor.value = project.bgColor
  useBgGradient.value = project.useBgGradient
  bgGradientColor.value = project.bgGradientColor
  borderColor.value = project.borderColor
  useBorderGradient.value = project.useBorderGradient
  borderGradientColor.value = project.borderGradientColor
  shadowColor.value = project.shadowColor
  useShadowGradient.value = project.useShadowGradient
  shadowGradientColor.value = project.shadowGradientColor
  textColor.value = project.textColor
  useTextGradient.value = project.useTextGradient
  textGradientColor.value = project.textGradientColor
  gradientDirection.value = project.gradientDirection
  showBorder.value = project.showBorder
  showShadow.value = project.showShadow
  selectedPreset.value = project.selectedPreset
  projectError.value = ''
  persistOpenedProject()

  if (fontImage.complete) {
    draw()
  }

  if (!silent) {
    setNotice(`Открыт проект «${project.name}»`, 'info')
  }
}

function deleteProject(projectId) {
  const project = savedProjects.value.find(item => item.id === projectId)
  if (!project) {
    return
  }

  const shouldDelete = window.confirm(`Удалить проект «${project.name}»?`)
  if (!shouldDelete) {
    return
  }

  savedProjects.value = savedProjects.value.filter(savedProject => savedProject.id !== projectId)

  if (openedProjectId.value === projectId) {
    openedProjectId.value = ''
    projectName.value = ''
    persistOpenedProject()
  }

  persistProjects()
  setNotice('Проект удалён', 'info')
}

function startNewProject() {
  openedProjectId.value = ''
  projectName.value = ''
  projectError.value = ''
  persistOpenedProject()
  setNotice('Создан новый черновик', 'info')
}

function formatProjectDate(isoDate) {
  return new Date(isoDate).toLocaleString('ru-RU')
}

function restoreProjectsFromStorage() {
  let parsedProjects = []

  try {
    const raw = localStorage.getItem(PROJECTS_STORAGE_KEY)
    if (raw) {
      const parsed = JSON.parse(raw)
      if (Array.isArray(parsed)) {
        parsedProjects = parsed.map(normalizeProject)
      }
    }
  } catch {
    parsedProjects = []
  }

  savedProjects.value = sortProjects(parsedProjects)

  const openedId = localStorage.getItem(OPENED_PROJECT_STORAGE_KEY)
  if (openedId) {
    openProject(openedId, true)
  }
}

function handleKeyboardShortcuts(event) {
  if (!event.metaKey && !event.ctrlKey) {
    return
  }

  const key = event.key.toLowerCase()

  if (key === 's') {
    event.preventDefault()
    saveProject()
    return
  }

  if (key === 'enter') {
    event.preventDefault()
    downloadImage()
  }
}

watch(
  [
    text,
    bgColor,
    useBgGradient,
    bgGradientColor,
    borderColor,
    useBorderGradient,
    borderGradientColor,
    shadowColor,
    useShadowGradient,
    shadowGradientColor,
    textColor,
    useTextGradient,
    textGradientColor,
    gradientDirection,
    showBorder,
    showShadow
  ],
  () => {
    if (fontImage.complete) draw()
  }
)

onMounted(() => {
  restoreProjectsFromStorage()

  fontImage.onerror = () => {
    errorText.value = `Не удалось загрузить шрифт: ${fontAssetPath}`
  }

  fontImage.onload = () => {
    errorText.value = ''
    analyzeCharWidths()
    draw()
  }

  window.addEventListener('keydown', handleKeyboardShortcuts)

  if (fontImage.complete) {
    analyzeCharWidths()
    draw()
  }
})

onBeforeUnmount(() => {
  if (noticeTimerId) {
    window.clearTimeout(noticeTimerId)
  }

  window.removeEventListener('keydown', handleKeyboardShortcuts)
})
</script>

<style>
:root {
  --bg-0: #020304;
  --bg-1: #0b0d11;
  --panel: #0f1217;
  --panel-soft: #12171d;
  --surface-1: rgba(11, 13, 17, 0.92);
  --surface-2: rgba(14, 17, 22, 0.97);
  --edge: #2b333d;
  --edge-soft: #242c35;
  --text-main: #ecf8ff;
  --text-muted: #95b4c8;
  --accent: #3bd6a9;
  --accent-strong: #86f4d5;
  --danger: #e77979;
  --chip: #0f1923;
}

* {
  box-sizing: border-box;
}

html,
body,
#app {
  margin: 0;
  width: 100%;
  min-height: 100%;
}

body {
  font-family: 'Sora', 'IBM Plex Sans', 'Manrope', 'Segoe UI', sans-serif;
  color: var(--text-main);
  background:
    radial-gradient(circle at 10% 10%, rgba(52, 85, 112, 0.18), transparent 38%),
    radial-gradient(circle at 84% 16%, rgba(40, 112, 94, 0.12), transparent 42%),
    linear-gradient(165deg, var(--bg-0), var(--bg-1));
  padding: 0;
}

.app {
  display: flex;
  flex-direction: column;
  gap: 0.95rem;
  width: min(1160px, 100%);
  margin: 0 auto;
  min-height: 100vh;
  padding: 1rem 1rem 1.1rem;
}

.editor-shell,
.projects-panel {
  border: 1px solid var(--edge);
  border-radius: 20px;
  background: linear-gradient(180deg, rgba(14, 18, 24, 0.96), rgba(16, 21, 28, 0.94));
  box-shadow: 0 16px 40px rgba(3, 8, 14, 0.34);
  backdrop-filter: blur(5px);
}

.editor-shell {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  padding: 1rem;
  animation: panel-rise 0.42s ease;
}

.projects-panel {
  display: flex;
  flex-direction: column;
  gap: 0.65rem;
  padding: 0.9rem;
  animation: panel-rise 0.5s ease;
}

.panel-block {
  border: 1px solid var(--edge-soft);
  border-radius: 14px;
  background: linear-gradient(180deg, var(--surface-1), var(--surface-2));
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.03);
}

.editor-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 1rem;
}

.editor-heading {
  display: flex;
  flex-direction: column;
  gap: 0.22rem;
}

.editor-kicker {
  margin: 0;
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 0.11em;
  font-size: 0.71rem;
}

.editor-title {
  margin: 0;
  font-size: clamp(1.35rem, 2vw, 1.62rem);
  line-height: 1.1;
}

.editor-subtitle {
  margin: 0;
  color: var(--text-muted);
  font-size: 0.83rem;
}

.header-actions {
  display: flex;
  gap: 0.55rem;
  flex-wrap: wrap;
  justify-content: flex-end;
}

.preview-section {
  display: flex;
  flex-direction: column;
  gap: 0.62rem;
  padding: 0.95rem;
}

.preview-topline {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
  align-items: center;
}

.preview-meta {
  margin: 0;
  color: var(--text-muted);
  font-size: 0.78rem;
  white-space: nowrap;
}

.text-input-row {
  display: flex;
  gap: 0.55rem;
}

.clear-button {
  flex: 0 0 auto;
}

.preview-canvas {
  display: none;
}

.preview-frame {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 136px;
  border: 1px solid var(--edge-soft);
  border-radius: 12px;
  background:
    repeating-linear-gradient(
      -45deg,
      rgba(255, 255, 255, 0.018) 0,
      rgba(255, 255, 255, 0.018) 9px,
      rgba(0, 0, 0, 0.08) 9px,
      rgba(0, 0, 0, 0.08) 18px
    );
  padding: 0.7rem;
}

.preview-img {
  max-width: 100%;
  min-height: 94px;
  border-radius: 8px;
  border: 1px solid var(--edge);
  background: #0a1118;
  image-rendering: pixelated;
  box-shadow: inset 0 0 0 1px rgba(255, 255, 255, 0.06);
}

.error-text {
  margin: 0;
  color: #ffa4a4;
  font-size: 0.84rem;
}

.notice-text {
  margin: 0;
  font-size: 0.82rem;
  padding: 0.4rem 0.55rem;
  border-radius: 8px;
  border: 1px solid transparent;
}

.notice-info {
  color: #a3d3ff;
  background: rgba(41, 97, 146, 0.22);
  border-color: rgba(104, 171, 231, 0.35);
}

.notice-success {
  color: #9ef4db;
  background: rgba(36, 138, 108, 0.23);
  border-color: rgba(116, 230, 196, 0.35);
}

.notice-error {
  color: #ffb0b0;
  background: rgba(140, 48, 48, 0.25);
  border-color: rgba(231, 121, 121, 0.35);
}

.settings-shell {
  display: grid;
  grid-template-columns: minmax(0, 1.58fr) minmax(240px, 0.9fr);
  gap: 0.75rem;
}

.settings-block {
  display: flex;
  flex-direction: column;
  gap: 0.65rem;
  padding: 0.95rem;
}

.settings-title {
  margin: 0;
  color: var(--text-muted);
  font-size: 0.82rem;
  text-transform: uppercase;
  letter-spacing: 0.08em;
}

.compact-block .stacked-fields {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.quick-actions {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.quick-actions .dark-button {
  flex: 1 1 132px;
}

.field-label {
  color: var(--text-muted);
  font-size: 0.8rem;
}

.input {
  width: 100%;
  padding: 0.56rem 0.68rem;
  font-size: 0.95rem;
}

.input.dark-input,
.dark-select {
  width: 100%;
  border: 1px solid var(--edge-soft);
  border-radius: 10px;
  background: #11161d;
  color: var(--text-main);
  outline: none;
}

.input.dark-input::placeholder {
  color: #7d9aad;
}

.input.dark-input:focus,
.dark-select:focus {
  border-color: var(--accent);
  box-shadow: 0 0 0 2px rgba(59, 214, 169, 0.18);
}

.color-pickers {
  display: flex;
  flex-direction: column;
  gap: 0.55rem;
}

.color-row {
  display: grid;
  grid-template-columns: 95px minmax(0, 1fr);
  gap: 0.45rem;
  align-items: start;
}

.row-title {
  color: var(--text-muted);
  font-size: 0.82rem;
  padding-top: 0.32rem;
}

.row-controls {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 0.45rem;
  min-width: 0;
}

.inline-toggle {
  display: inline-flex;
  align-items: center;
  gap: 0.3rem;
  padding: 0.27rem 0.52rem;
  border-radius: 999px;
  border: 1px solid var(--edge-soft);
  background: var(--chip);
  color: var(--text-muted);
  line-height: 1;
  font-size: 0.77rem;
  user-select: none;
}

.inline-toggle input {
  margin: 0;
  accent-color: var(--accent);
}

.color-picker {
  width: 35px;
  height: 30px;
  padding: 0;
  border-radius: 8px;
  border: 1px solid var(--edge-soft);
  background: #11161d;
  cursor: pointer;
}

button {
  font-family: inherit;
}

.dark-button {
  appearance: none;
  border: 1px solid var(--edge);
  background: linear-gradient(180deg, #202832, #191f27);
  color: var(--text-main);
  border-radius: 10px;
  padding: 0.58rem 0.95rem;
  font-size: 0.84rem;
  font-weight: 600;
  cursor: pointer;
  white-space: nowrap;
  transition: transform 0.08s ease, border-color 0.18s ease, filter 0.18s ease;
}

.dark-button:hover:not(:disabled) {
  border-color: var(--accent);
  filter: brightness(1.08);
}

.dark-button:active:not(:disabled) {
  transform: translateY(1px);
}

.dark-button:disabled {
  cursor: not-allowed;
  opacity: 0.54;
  filter: saturate(0.65);
}

.primary-button {
  border-color: rgba(134, 244, 213, 0.5);
  background: linear-gradient(180deg, #234036, #1a3029);
}

.ghost-button {
  background: linear-gradient(180deg, #1d232c, #171c24);
}

.subtle-button {
  background: linear-gradient(180deg, #232a33, #1b212a);
}

.danger-button {
  border-color: #a05151;
  background: linear-gradient(180deg, #7a3a3a, #622f2f);
}

.danger-button:hover:not(:disabled) {
  border-color: var(--danger);
}

.projects-header {
  display: flex;
  justify-content: space-between;
  gap: 0.75rem;
  align-items: flex-start;
  padding: 0.88rem 0.95rem;
}

.projects-controls {
  display: flex;
  flex-direction: column;
  gap: 0.55rem;
  padding: 0.88rem 0.95rem;
}

.projects-list-shell {
  display: flex;
  flex-direction: column;
  min-height: 0;
  padding: 0.75rem;
  max-height: clamp(220px, 34vh, 360px);
}

.projects-title {
  margin: 0;
  font-size: 1.14rem;
}

.projects-subtitle {
  margin: 0.2rem 0 0;
  color: var(--text-muted);
  font-size: 0.78rem;
}

.projects-count {
  margin: 0;
  color: var(--accent-strong);
  font-size: 0.8rem;
  border: 1px solid rgba(120, 229, 197, 0.28);
  border-radius: 999px;
  padding: 0.22rem 0.52rem;
  background: rgba(39, 130, 108, 0.16);
}

.project-name-input,
.project-search-input {
  width: 100%;
}

.projects-toolbar {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.projects-toolbar .dark-button {
  flex: 1 1 132px;
}

.projects-list {
  display: flex;
  flex-direction: column;
  gap: 0.62rem;
  overflow: auto;
  min-height: 0;
  flex: 1;
  padding-right: 0.18rem;
}

.projects-list::-webkit-scrollbar {
  width: 8px;
}

.projects-list::-webkit-scrollbar-thumb {
  background: rgba(118, 152, 173, 0.35);
  border-radius: 99px;
}

.projects-empty {
  margin: 0;
  color: var(--text-muted);
  font-size: 0.85rem;
}

.project-card {
  display: grid;
  grid-template-columns: 116px minmax(0, 1fr);
  gap: 0.7rem;
  padding: 0.65rem;
  border-radius: 10px;
  border: 1px solid var(--edge-soft);
  background: linear-gradient(180deg, rgba(15, 19, 25, 0.95), rgba(13, 17, 22, 0.95));
  animation: card-rise 0.35s ease both;
}

.project-card-opened {
  border-color: rgba(134, 244, 213, 0.5);
  box-shadow: 0 0 0 1px rgba(134, 244, 213, 0.28) inset;
}

.project-preview {
  width: 116px;
  height: 56px;
  border: 1px solid var(--edge);
  border-radius: 6px;
  object-fit: contain;
  image-rendering: pixelated;
  background: #0a1118;
}

.project-preview-empty {
  display: flex;
  align-items: center;
  justify-content: center;
  color: #7d9aad;
  font-size: 0.78rem;
}

.project-info {
  display: flex;
  flex-direction: column;
  gap: 0.15rem;
  min-width: 0;
}

.project-name {
  margin: 0;
  font-size: 0.95rem;
  font-weight: 600;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.project-meta {
  margin: 0;
  color: var(--text-muted);
  font-size: 0.77rem;
}

.project-opened {
  margin: 0.16rem 0 0;
  color: var(--accent-strong);
  font-size: 0.77rem;
}

.project-actions {
  grid-column: 1 / -1;
  display: flex;
  justify-content: flex-end;
  gap: 0.45rem;
}

.project-actions .dark-button {
  flex: 0 0 auto;
}

@keyframes panel-rise {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes card-rise {
  from {
    opacity: 0;
    transform: translateY(8px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (max-width: 1160px) {
  .app {
    padding: 0.9rem;
  }

  .projects-list-shell {
    max-height: 320px;
  }
}

@media (max-width: 900px) {
  .app {
    padding: 0.82rem;
  }

  .settings-shell {
    grid-template-columns: 1fr;
  }

  .projects-header {
    align-items: center;
  }

  .projects-list-shell {
    max-height: 300px;
  }
}

@media (max-width: 640px) {
  .editor-shell,
  .projects-panel {
    padding: 0.82rem;
    border-radius: 14px;
  }

  .editor-header {
    flex-direction: column;
    align-items: stretch;
  }

  .header-actions,
  .projects-toolbar {
    width: 100%;
  }

  .header-actions .dark-button,
  .projects-toolbar .dark-button {
    flex: 1 1 100%;
  }

  .text-input-row {
    flex-direction: column;
  }

  .color-row {
    grid-template-columns: 1fr;
  }

  .row-title {
    padding-top: 0;
  }

  .project-card {
    grid-template-columns: 1fr;
  }

  .project-preview {
    width: 100%;
    height: 66px;
  }

  .project-actions {
    justify-content: stretch;
  }

  .project-actions .dark-button {
    flex: 1;
  }

  .preview-topline {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.25rem;
  }

  .projects-list-shell {
    max-height: 280px;
  }
}
</style>
