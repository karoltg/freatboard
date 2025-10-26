<template>
  <div class="w-full min-h-screen  text-gray-900 p-4">
    <div class="max-w-7xl mx-auto grid grid-cols-1 lg:grid-cols-7 gap-6">
      <!-- Controls -->
      <aside class="lg:col-span-2 bg-white rounded-2xl shadow p-4 space-y-4">
        <h2 class="text-xl font-bold">Settings</h2>
        <div>
        <div class="grid grid-cols-2 gap-3">
          <label class="text-sm">Number of freats
            <input type="number" min="3" max="30" v-model.number="numFrets"
              class="mt-1 w-full border rounded-xl px-3 py-2" />
          </label>
        </div>
        <div>
          <label class="text-sm">Tune preset
            <select v-model="preset"
              class="mt-1 w-full border rounded-xl px-3 py-2">
              <option v-for="(val, key) in DEFAULT_TUNINGS" :key="key" :value="key">{{ key }}</option>
            </select>
          </label>
        </div>
        </div>
        <div>
          <p class="text-sm font-medium mb-2">Strings</p>
          <div class="grid grid-cols-3 gap-2">
            <input
              v-for="(s, i) in strings" :key="i"
              v-model="strings[i]"
              class="border rounded-xl px-2 py-1"
            />
          </div>
          <div class="mt-2 flex gap-2 flex-wrap">
            <button
              class="px-3 py-2 rounded-xl bg-gray-100 hover:bg-gray-200"
              @click="addStringTop"
              title="Dodaj strunę na górze"
            >+ struna</button>
            <button
              class="px-3 py-2 rounded-xl bg-gray-100 hover:bg-gray-200"
              @click="removeStringTop"
              title="Usuń górną strunę"
            >– struna</button>


          </div>
        </div>

        <div class="grid grid-cols-2 gap-3">
          <!--na razie zakomentowane dla leworęcznosci bo wymaga poprawek w wyswietlaniu gryfu-->
          <!--<label class="inline-flex items-center gap-2 text-sm">-->
          <!--  <input type="checkbox" v-model="lefty" />-->
          <!--  Leworęczne (odwróć)-->
          <!--</label>-->
          <label class="inline-flex items-center gap-2 text-sm">
            <input type="checkbox" v-model="showFretNums" />
            Freat Number
          </label>
          <label class="inline-flex items-center gap-2 text-sm">
            <input type="checkbox" v-model="showInlays" />
            Freatboard sign
          </label>
          <label class="inline-flex items-center gap-2 text-sm">
            <input type="checkbox" v-model="autoLabelNote" />
            Auto-label: note
          </label>
        </div>
        <div class="border-t pt-4 flex flex-wrap gap-2">
          <button class="px-3 py-2 rounded-xl bg-indigo-600 text-white hover:bg-indigo-700" @click="downloadSVG">Download SVG</button>
          <button class="px-3 py-2 rounded-xl bg-teal-600 text-white hover:bg-teal-700" @click="downloadPNG(2)">Download PNG</button>
        </div>

        <p class="text-xs text-gray-500 pt-2">Tips: Click on the grid (intersection of the string and fret) to add a marker. Enable “Auto-label: note” to insert note names according to the tuning.</p>

      </aside>

      <!-- Canvas -->
      <main class="lg:col-span-5 bg-white rounded-2xl shadow p-4 overflow-auto">
        <div class="flex items-center justify-between mb-2">
          <h2 class="text-xl font-bold">Fretboard</h2>
          <span class="text-sm text-gray-500">{{ stringCount }} strings • {{ fretCount }} freats</span>
        </div>

        <div class="w-full overflow-auto">
          <svg
            ref="svgRef"
            :width="totalW"
            :height="totalH + (showFretNums ? 20 : 0)"
            :viewBox="`0 0 ${totalW} ${totalH + (showFretNums ? 20 : 0)}`"
            class="border rounded-xl bg-[#fdfdfd] cursor-crosshair"
            @click="handleBoardClick"
          >
            <!-- Background -->
            <rect :x="0" :y="0" :width="totalW" :height="totalH + (showFretNums ? 20 : 0)" fill="#f8fafc" />

            <!-- Fretboard wood -->
            <rect
              :x="boardPad.x"
              :y="boardPad.y - 18"
              :width="nutW + cellW * (fretCount + 1)"
              :height="cellH * (stringCount - 1) + 36"
              rx="16"
              fill="#e5e7eb"
              stroke="#d1d5db"
            />

            <!-- Strings -->
            <line
              v-for="i in stringCount"
              :key="`string-${i-1}`"
              :x1="boardPad.x"
              :y1="yForString(i-1)"
              :x2="boardPad.x + zeroW + nutW + cellW * fretCount"
              :y2="yForString(i-1)"
              stroke="#111827"
              :stroke-width="(i-1) === 0 || (i-1) === stringCount - 1 ? 2.5 : 2"
              stroke-linecap="round"
            />

            <!-- Frets (nut + kolejne) przesunięte o zeroW w prawo -->
            <line
              v-for="f in fretCount + 1"
              :key="`fret-${f-1}`"
              :x1="boardPad.x + zeroW + nutW + (f-1) * cellW"
              :y1="boardPad.y - 14"
              :x2="boardPad.x + zeroW + nutW + (f-1) * cellW"
              :y2="boardPad.y + cellH * (stringCount - 1) + 14"
              stroke="#6b7280"
              :stroke-width="(f-1) === 0 ? 6 : 2"
            />

            <!-- Inlays -->
            <template v-if="showInlays">
              <template v-for="fret in inlayFrets" :key="`inlay-${fret}`">
                <template v-if="fret === 12 || fret === 24">
                  <circle :cx="boardPad.x + zeroW + nutW + (fret - 0.5) * cellW" :cy="yMid - 10" r="5" fill="#9ca3af" />
                  <circle :cx="boardPad.x + zeroW + nutW + (fret - 0.5) * cellW" :cy="yMid + 10" r="5" fill="#9ca3af" />
                </template>
                <template v-else>
                  <circle :cx="boardPad.x + zeroW + nutW + (fret - 0.5) * cellW" :cy="yMid" r="5" fill="#9ca3af" />
                </template>
              </template>
            </template>

            <!-- Markery -->
            <g v-for="m in markerList" :key="`mk-${m.s}-${m.f}`">
              <circle :cx="markerCx(m)" :cy="markerCy(m)" :r="m.r" :fill="m.fill" stroke="#000" :stroke-opacity="0.25" />
              <text v-if="m.label" :x="markerCx(m)" :y="markerCy(m) + 4" font-size="12" font-weight="600" text-anchor="middle" :fill="m.textColor">
                {{ m.label }}
              </text>
            </g>

            <!-- Fret numbers: 0 po lewej + 1..N pod deską -->
            <template v-if="showFretNums">
              <text :x="boardPad.x + zeroW / 2" :y="totalH + 10" text-anchor="middle" font-size="18" fill="#374151">0</text>
              <text v-for="f in fretCount" :key="`fnum-${f}`" :x="boardPad.x + zeroW + nutW + (f - 0.5) * cellW" :y="totalH + 10" text-anchor="middle" font-size="18" fill="#374151">
                {{ lefty ? fretCount - f + 1 : f }}
              </text>
            </template>
          </svg>
        </div>

        <div class="max-w-7xl mx-auto grid grid-cols-1 lg:grid-cols-3 gap-3">
  <!-- Kolumna 1: Marker — tool + Select color -->
  <div class="border-t pt-4 space-y-4">
    <h3 class="font-semibold">Marker — tool</h3>

    <div class="grid grid-cols-2 gap-3">
      <!--<label class="text-sm">Tekst etykiety
        <input v-model="markerText"
          class="mt-1 w-full border rounded-xl px-3 py-2" placeholder="np. R, 3, #" />
      </label>-->

      <div class="flex items-center gap-2 col-span-2">
        <button
          class="w-8 h-8 rounded-full border border-gray-300 hover:ring"
          :style="{ backgroundColor: '#2563eb' }"
          title="Niebieski"
          @click="setPresetFill('#2563eb')"
        />
        <button
          class="w-8 h-8 rounded-full border border-gray-300 hover:ring"
          :style="{ backgroundColor: '#84cc16' }"
          title="Limonkowy"
          @click="setPresetFill('#84cc16')"
        />
        <button
          class="w-8 h-8 rounded-full border border-gray-300 hover:ring"
          :style="{ backgroundColor: '#f59e0b' }"
          title="Pomarańczowy"
          @click="setPresetFill('#f59e0b')"
        />
      </div>
      <label class="text-sm">Fill color
        <input type="color" v-model="markerFill"
               class="mt-1 w-full border rounded-xl px-3 py-2 h-10" />
      </label>

      <label class="text-sm">Text color
        <input type="color" v-model="markerTextColor"
               class="mt-1 w-full border rounded-xl px-3 py-2 h-10" />
      </label>
    </div>
    <label class="text-sm">Radius (px)
      <input type="number" min="6" max="24" v-model.number="markerRadius"
             class="mt-1 w-full border rounded-xl px-3 py-2" />
    </label>

    <div class="flex gap-2">
      <button class="px-3 py-2 rounded-xl bg-gray-100 hover:bg-gray-200"
              @click="resetMarkers">Clean markers</button>
    </div>
    <!-- Select color (presety) przeniesione pod Marker — tool -->
    <div class="pt-2">
      <label class="block text-sm font-medium mb-2">Select color (presets)</label>

    </div>
  </div>

  <!-- Kolumna 2: placeholder -->
  <div class="border-t space-y-2">
    <!-- Kolumna 2: Akordy / zaznaczanie stopni -->
    <div class="border-t pt-4 space-y-4">
      <h3 class="font-semibold">Grid 2 — Chord highlight</h3>

      <div class="grid grid-cols-2 gap-3">
        <label class="text-sm col-span-2">Chord preset
          <select v-model="selectedChord"
                  class="mt-1 w-full border rounded-xl px-3 py-2">
            <option v-for="(v, k) in CHORD_PRESETS" :key="k" :value="k as any">
              {{ k }}
            </option>
          </select>
        </label>

        <!-- Legenda stopni (dynamiczna) -->
        <div class="col-span-2 flex flex-wrap items-center gap-2">
          <template v-for="item in chordLegend" :key="item.deg">
            <span
              class="inline-flex items-center gap-2 text-xs px-2 py-1 rounded-full border"
              :style="{ backgroundColor: item.color, color: getTextColorForBg(item.color) }"
            >
              <span class="font-semibold">{{ item.deg }}</span>
              <span class="opacity-90">— {{ item.notes.join(' / ') }}</span>
            </span>
          </template>
        </div>


        <div class="col-span-2 flex gap-2">
          <button class="px-3 py-2 rounded-xl bg-gray-100 hover:bg-gray-200"
                  @click="addChordMarkers">Highlight chord</button>
          <button class="px-3 py-2 rounded-xl bg-gray-100 hover:bg-gray-200"
                  @click="clearChordMarkers">Clear chord</button>
        </div>

        <p class="col-span-2 text-xs text-gray-500">
          Podświetla wszystkie dźwięki wybranego akordu na całym gryfie. Ręczne markery nie są nadpisywane.
        </p>
      </div>
    </div>

  </div>

  <!-- Kolumna 3: placeholder -->
  <div class="border-t pt-4 space-y-2">
    <h3 class="font-semibold">Grid 3</h3>
    <!-- Twoje kontrolki tutaj -->
    <div class="text-sm text-gray-500">Miejsce na dodatkowe ustawienia…</div>
  </div>
</div>


      </main>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed, watch } from 'vue'
import UiButton from '../components/UiButton.vue'


const DEFAULT_TUNINGS = {
  'Guitar E Standard (6)': ['E2', 'A2', 'D3', 'G3', 'B3', 'E4'],
  'Guitar D Standard (6)': ['D2', 'G2', 'C3', 'F3', 'A3', 'D4'],
  'Guitar Drop D (6)': ['D2', 'A2', 'D3', 'G3', 'B3', 'E4'],
  'Bass E Standard (4)': ['E1', 'A1', 'D2', 'G2'],
  'Ukulele C (4)': ['G4', 'C4', 'E4', 'A4']
} as const

type PresetKey = keyof typeof DEFAULT_TUNINGS

const NOTE_NAMES = ['C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B']

type Marker = {
  s: number
  f: number // 0 = otwarta struna; 1..N = progi
  label: string
  fill: string
  textColor: string
  r: number
  kind?: 'manual' | 'chord' // znacznik, czy ręcznie czy akord jest dodawany
}

function yForString(s: number) {
  // s = indeks logiczny struny (0 = najgrubsza)
  return boardPad.y + (stringCount.value - 1 - s) * cellH
}

function noteNameToMidi(n: string) {
  const m = n.match(/^([A-G])(#?)(\d)$/);
  if (!m) return null;
  const [, letter, sharpSign, octaveStr] = m as RegExpMatchArray;
  if (!letter || !octaveStr) return null;
  const baseMap = { C: 0, D: 2, E: 4, F: 5, G: 7, A: 9, B: 11 } as const;
  const base = baseMap[letter as keyof typeof baseMap];
  const sharp = sharpSign === "#" ? 1 : 0;
  const octave = Number.parseInt(octaveStr, 10);
  return 12 * (octave + 1) + base + sharp;
}

function midiToNoteName(m: number) {
  const n = ((m % 12) + 12) % 12;
  return NOTE_NAMES[n];
}

// State
const numFrets = ref(12)
const strings = ref<string[]>([...DEFAULT_TUNINGS['Guitar E Standard (6)']!])
const preset = ref<PresetKey>('Guitar E Standard (6)')
const lefty = ref(false)
const showFretNums = ref(true)
const showInlays = ref(true)
const eraseMode = ref(false)
const autoLabelNote = ref(true)

// Marker tool
const markerFill = ref('#111827')
const markerText = ref('')
const markerTextColor = ref('#ffffff')
const markerRadius = ref(14)

// Markers
const markers = reactive<Record<string, Marker>>({})

// Layout constants
const cellW = 54
const cellH = 44
const nutW = 12
const boardPad = { x: 24, y: 24 }
// Szerokość obszaru progu 0 (otwarta struna) po lewej stronie
const zeroW = cellW

const stringCount = computed(() => strings.value.length)
const fretCount = computed(() => Math.max(1, numFrets.value))
const totalW = computed(() => boardPad.x * 2 + zeroW + nutW + cellW * fretCount.value)
const totalH = computed(() => boardPad.y * 2 + cellH * (stringCount.value - 1))
const yMid = computed(() => boardPad.y + (cellH * (stringCount.value - 1)) / 2)

const inlayFrets = computed(() => {
  const list = [3, 5, 7, 9, 12, 15, 17, 19, 21, 24]
  return list.filter((f) => f <= fretCount.value)
})

const svgRef = ref<SVGSVGElement | null>(null)

watch(preset, (p) => {
  strings.value = [...DEFAULT_TUNINGS[p]]
})

function noteAt(s: number, f: number) {
  const tuning = strings.value[s];
  if (!tuning) return '';
  const midi = noteNameToMidi(tuning);
  if (midi == null) return '';
  return midiToNoteName(midi + f);
}

function toggleMarker(s: number, f: number) {
  const key = `${s}-${f}`
  if (eraseMode.value) {
    delete markers[key]
    return
  }
  const baseLabel = autoLabelNote.value ? noteAt(s, f) : markerText.value
  if (markers[key]) {
    delete markers[key]
  } else {
    markers[key] = {
      s,
      f,
      label: baseLabel || '',
      fill: markerFill.value,
      textColor: markerTextColor.value,
      r: markerRadius.value,
      kind: 'manual'
    }
  }
}

function handleBoardClick(evt: MouseEvent) {
  const svg = svgRef.value
  if (!svg) return
  const pt = svg.createSVGPoint()
  pt.x = (evt as MouseEvent).clientX
  pt.y = (evt as MouseEvent).clientY
  const ctm = svg.getScreenCTM()?.inverse()
  if (!ctm) return
  const loc = pt.matrixTransform(ctm)

  const xStart = boardPad.x
  const yStart = boardPad.y
  const xr = loc.x - xStart
  const yr = loc.y - yStart

  if (xr < 0 || yr < -cellH / 2) return

  // f = 0 w strefie zeroW lub na siodełku; w przeciwnym razie 1..fretCount
  let f = 0
  if (xr >= zeroW + nutW) {
    const afterNut = xr - zeroW - nutW
    f = Math.floor(afterNut / cellW) + 1
    if (f < 1) f = 1
    if (f > fretCount.value) f = fretCount.value
  }

  let s = Math.round(yr / cellH)
  if (s < 0) s = 0
  if (s > stringCount.value - 1) s = stringCount.value - 1

  const logicalS = lefty.value ? stringCount.value - 1 - s : s
  const logicalF = f === 0 ? 0 : (lefty.value ? (fretCount.value - f + 1) : f)

  toggleMarker(logicalS, logicalF)
}

function downloadSVG() {
  const svg = svgRef.value
  if (!svg) return
  const serializer = new XMLSerializer()
  const src = serializer.serializeToString(svg)
  const blob = new Blob([src], { type: 'image/svg+xml;charset=utf-8' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'fretboard.svg'
  a.click()
  URL.revokeObjectURL(url)
}

function downloadPNG(scale = 2) {
  const svgEl = svgRef.value
  if (!svgEl) return
  const serializer = new XMLSerializer()
  const svgString = serializer.serializeToString(svgEl)
  const svgBlob = new Blob([svgString], { type: 'image/svg+xml;charset=utf-8' })
  const url = URL.createObjectURL(svgBlob)

  const img = new Image()
  img.onload = () => {
    const canvas = document.createElement('canvas')
    canvas.width = img.width * scale
    canvas.height = img.height * scale
    const ctx = canvas.getContext('2d')!
    ctx.setTransform(scale, 0, 0, scale, 0, 0)
    ctx.drawImage(img, 0, 0)
    canvas.toBlob((blob) => {
      if (!blob) return
      const a = document.createElement('a')
      a.href = URL.createObjectURL(blob)
      a.download = 'fretboard.png'
      a.click()
      URL.revokeObjectURL(a.href)
      URL.revokeObjectURL(url)
    })
  }
  img.src = url
}

function addStringTop() {
  strings.value = [strings.value[0]!, ...strings.value];
}

function removeStringTop() {
  if (strings.value.length > 1) strings.value = strings.value.slice(1)
}

function resetMarkers() {
  for (const k of Object.keys(markers)) delete markers[k]
}

const markerList = computed<Marker[]>(() => Object.values(markers))

function markerCx(m: Marker) {
  if (m.f === 0) {
    // próg 0 zawsze po lewej – środek strefy zeroW
    return boardPad.x + zeroW / 2
  }
  // dla progów 1..N zachowujemy odbicie dla trybu leworęcznego
  const fDraw = lefty.value ? (fretCount.value - m.f + 1) : m.f
  return boardPad.x + zeroW + nutW + (fDraw - 0.5) * cellW
}

function markerCy(m: Marker) {
//  const s = lefty.value ? stringCount.value - 1 - m.s : m.s
//  return boardPad.y + s * cellH
return yForString(m.s)
}

// Prosty kontrast: dobierz kolor tekstu (biały/czarny) do tła markera
function getTextColorForBg(hex: string) {
  // akceptujemy #rgb lub #rrggbb
  const c = hex.replace('#','')
  const r = c.length === 3 ? parseInt(c[0]+c[0], 16) : parseInt(c.slice(0,2), 16)
  const g = c.length === 3 ? parseInt(c[1]+c[1], 16) : parseInt(c.slice(2,4), 16)
  const b = c.length === 3 ? parseInt(c[2]+c[2], 16) : parseInt(c.slice(4,6), 16)
  // luminancja względna (upraszczamy do luma sRGB)
  const luma = 0.2126*r + 0.7152*g + 0.0722*b
  return luma > 140 ? '#111827' /* ciemny */ : '#ffffff' /* jasny */
}

function setPresetFill(color: string) {
  markerFill.value = color
  // Jeśli użytkownik nie nadpisuje ręcznie, ustaw czytelny tekst:
  markerTextColor.value = getTextColorForBg(color)
}


// Kolory stopni akordu (R = pryma)
const DEGREE_COLORS: Record<string, string> = {
  '1':  '#ef4444',
  '3':  '#10b981',
  '5':  '#3b82f6',
  '7':  '#f59e0b',     // maj7
  'b7': '#f97316',     // dom.7
  '9':  '#a855f7',     // 9
  'b9': '#dc2626',     // b9
  '#9': '#06b6d4',     // #9
}


// Presety akordów (na start C dur i Cmaj7)
const CHORD_PRESETS = {
  // już miałeś:
  'C major (C E G)': {
    degrees: { C: '1', E: '3', G: '5' }
  },
  'Cmaj7 (C E G B)': {
    degrees: { C: '1', E: '3', G: '5', B: '7' }
  },

  // NOWE: z 9
  'Cadd9 (C E G D)': {
    degrees: { C: '1', E: '3', G: '5', D: '9' }
  },
  'C9 (C E G Bb D)': {
    // Bb -> A#
    degrees: { C: '1', E: '3', G: '5', 'A#': 'b7', D: '9' }
  },
  'Cmaj9 (C E G B D)': {
    degrees: { C: '1', E: '3', G: '5', B: '7', D: '9' }
  },

  // Opcjonalnie alteracje 9:
  'C7(b9) (C E G Bb Db)': {
    // Db -> C#
    degrees: { C: '1', E: '3', G: '5', 'A#': 'b7', 'C#': 'b9' }
  },
  'C7(#9) (C E G Bb D#)': {
    degrees: { C: '1', E: '3', G: '5', 'A#': 'b7', 'D#': '#9' }
  },
} as const


type ChordKey = keyof typeof CHORD_PRESETS

const selectedChord = ref<ChordKey>('C major (C E G)')


function clearChordMarkers() {
  for (const k of Object.keys(markers)) {
    if (markers[k]?.kind === 'chord') delete markers[k]
  }
}

function addChordMarkers() {
  const preset = CHORD_PRESETS[selectedChord.value]
  if (!preset) return
  clearChordMarkers()

  for (let s = 0; s < stringCount.value; s++) {
    for (let f = 0; f <= fretCount.value; f++) {
      const n = noteAt(s, f) // np. 'C', 'C#', ...
      const deg = (preset.degrees as Record<string, string>)[n]
      if (!deg) continue

      const fill = DEGREE_COLORS[deg] || '#111827'
      const key = `${s}-${f}`

      if (markers[key]?.kind === 'manual') continue

      markers[key] = {
        s, f,
        label: n,                         // ⬅️ ZAMIANA: pokazujemy NUTĘ (C/E/G/B), jak zamienimy na deg to pokaże numer dźwięku w akordzie a jak n to nutę np C
        fill,
        textColor: getTextColorForBg(fill),
        r: markerRadius.value,
        kind: 'chord'
      }
    }
  }
}

// Kolejność sortowania stopni w legendzie (opcjonalnie rozszerz)
const DEGREE_ORDER = ['1','b2','2','#2','b3','3','4','#4','b5','5','#5','b6','6','bb7','b7','7','b9','9','#9','11','#11','b13','13']

function degreeRank(d: string) {
  const i = DEGREE_ORDER.indexOf(d)
  return i === -1 ? 999 : i
}

// Zbuduj dynamiczną legendę: [{ deg: 'R', notes: ['C'], color: '#...' }, ...]
const chordLegend = computed(() => {
  const preset = CHORD_PRESETS[selectedChord.value]
  if (!preset) return []

  // mapa: stopień -> lista nut
  const byDegree: Record<string, string[]> = {}
  for (const [note, deg] of Object.entries(preset.degrees)) {
    if (!byDegree[deg]) byDegree[deg] = []
    byDegree[deg].push(note)
  }

  const items = Object.entries(byDegree).map(([deg, notes]) => ({
    deg,
    notes,
    color: DEGREE_COLORS[deg] || '#111827',
  }))

  // posortuj wg DEGREE_ORDER
  items.sort((a, b) => degreeRank(a.deg) - degreeRank(b.deg))
  return items
})



</script>

<style scoped>
/* brak dodatkowych styli — Tailwind wystarcza */
</style>
