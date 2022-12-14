<script setup lang="ts">
import { onMounted } from '@vue/runtime-core'
import { computed, ref } from 'vue'
import { useNow } from '@vueuse/core'
import fetchJsonp from 'fetch-jsonp'
import {
  VFetch,
  debounce,
  findElement,
  getDevice,
  getLocation,
  removeElement,
  useEventListener,
} from 'lazy-js-utils'
import gitForkVue from '@simon_he/git-fork-vue'
// base
const data = [
  {
    src: new URL('../../public/1.png', import.meta.url).href,
    color: '#ff1100',
  },
  {
    src: new URL('../../public/2.png', import.meta.url).href,
    color: '#ff8800',
  },
  { src: new URL('../../public/3.png', import.meta.url).href, color: 'green' },
  {
    src: new URL('../../public/4.png', import.meta.url).href,
    color: '#779922',
  },
  { src: new URL('../../public/5.png', import.meta.url).href, color: 'blue' },
  {
    src: new URL('../../public/6.png', import.meta.url).href,
    color: '#335577',
  },
  {
    src: new URL('../../public/7.png', import.meta.url).href,
    color: '#441122',
  },
  {
    src: new URL('../../public/8.png', import.meta.url).href,
    color: '#216644',
  },
  {
    src: new URL('../../public/9.png', import.meta.url).href,
    color: '#771199',
  },
]
// card size
const { os } = getDevice()
const getSize = () => {
  if (os === 'ios' || os === 'android')
    return 35
  return 60
}
const size = getSize()

const rows = 10
const cols = 10
// eliminate the number of cards
const eliminateCount = 3
// group
const group = 6
// layer
const layerCount = 6
const cellHtml: string[] = []
const city = ref('未知位置')
getLocation().then(async (res) => {
  if (!res)
    return
  const { latitude, longitude } = res
  const { json } = await fetchJsonp(
    `https://api.map.baidu.com/geocoder/v2/?ak=PQRexdBQlNuCemC80ziGPITLa8FOuThD&output=json&pois=1&location=${latitude},${longitude}`,
  )
  const { result } = await json()
  const c = result?.addressComponent?.city
  if (city.value)
    city.value = c
})
let moveList: HTMLElement
let main: HTMLElement
let gameOver = false
const now = useNow()
const endTime = ref<number>(0)
const startTime = ref<number>(0)
const countDown = computed(() =>
  startTime.value
    ? Math.round(((endTime.value || +now.value) - startTime.value) / 1000)
    : 0,
)
const renderData = Array.from({ length: eliminateCount * group }, (_, i) =>
  data.map(v => ({ ...v })),
)
  .flat()
  .sort(() => Math.random() - 0.5)
for (let ly = layerCount - 1; ly >= 0; ly--) {
  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      const pyStep = (ly + 1) % 2 === 0 ? size / 2 : 0
      const item = Math.random() > 0.7 && renderData.pop()
      if (item) {
        cellHtml.push(
          `<div class="item" id="m${ly}-${i}-${j}" style="width:${size}px;height:${size}px;left:${
            size * j + pyStep
          }px;top:${size * i + pyStep}px;background-color:${item.color}"><img src="${
            item.src
          }"/></div>`,
        )
      }
    }
  }
}
const checkDisabled = () => {
  findElement('.item', true)?.forEach((v) => {
    const arr = v.id.substring(1).split('-').map(Number)
    const [x, y, z] = arr
    const isPy = (x + 1) % 2 === 0
    const i = x + 1
    const isPyB = (i + 1) % 2 === 0
    if (isPy === isPyB) {
      const el = findElement(`#m${i}-${y}-${z}`, main)
      if (el)
        v.classList.add('disabled')
    }
    else if (isPy && !isPyB) {
      const result = [
        `${i}-${y}-${z}`,
        `${i}-${y}-${z + 1}`,
        `${i}-${y + 1}-${z}`,
        `${i}-${y + 1}-${z + 1}`,
      ].every(k => !findElement(`#m${k}`, main))
      if (!result)
        v.classList.add('disabled')
      else v.classList.remove('disabled')
    }
    else if (!isPy && isPyB) {
      const result = [
        `${i}-${y}-${z}`,
        `${i}-${y}-${z - 1}`,
        `${i}-${y - 1}-${z}`,
        `${i}-${y - 1}-${z - 1}`,
      ].every(k => !findElement(`#m${k}`, main))
      if (!result)
        v.classList.add('disabled')
      else v.classList.remove('disabled')
    }
  })
}
const setDisabled = debounce(checkDisabled, 50)
onMounted(() => {
  main = findElement('.main')!
  moveList = findElement('.move-list')!
  main.innerHTML = cellHtml.reverse().join('')
  main.style.height = `${size * rows + size * 2}px`
  main.style.width = `${size * cols}px`
  moveList.style.height = `${size}px`
  moveList.style.width = `${size * group}px`
  setDisabled()
})

const saveGameData = () => {
  const axios = new VFetch({})
  axios
    .get({
      url: `http://api.car_by_car.hejian.club/rank?city=${city.value}&times=${countDown.value}`,
    })
    .then((index) => {
      alert(`恭喜你！你当前是${city.value},第${++index}名`)
    })
    .catch(() => {
      alert('You win')
    })
}

const move = (e: MouseEvent) => {
  if (!startTime.value)
    startTime.value = Date.now()
  if (gameOver)
    return
  const target = e.target as HTMLElement
  if (!target.className.includes('item'))
    return
  const top = moveList.offsetTop - main.offsetTop
  if (target.className.includes('disabled'))
    return

  const left = (main.offsetWidth - moveList.offsetWidth) / 2
  const len = moveList.children.length
  target.style.top = `${top}px`
  target.style.left = `${len * size + left}px`
  let count = 0
  const eliminateCheck = () => {
    const moveListChildren = Array.from(moveList.children).filter(
      v => v.innerHTML === target.innerHTML,
    ) as HTMLElement[]
    if (moveListChildren.length === 3) {
      return moveListChildren.forEach((v) => {
        v.style.transform = 'scale(0)'
        setTimeout(() => {
          removeElement(v)
        }, 300)
      })
    }
    if (moveList.children.length === group) {
      gameOver = true
      endTime.value = (now.value as unknown) as number
      return alert('You lose!')
    }
  }
  const stop = useEventListener(target, 'transitionend', () => {
    count++
    if (count === 2) {
      stop()
      target.style.top = '0px'
      target.style.left = '0px'
      target.style.position = 'relative'
      target.style.display = 'inline-flex'
      const newTarget = target.cloneNode(true) as HTMLElement
      if (gameOver)
        return
      moveList.appendChild(newTarget)
      eliminateCheck()
      removeElement(target)
      const resetChildren = main.children
      if (resetChildren.length === 0) {
        gameOver = true
        endTime.value = (now.value as unknown) as number
        return saveGameData()
      }
      setDisabled()
    }
  })
}
</script>

<template>
  <gitForkVue
    link="https://github.com/Simon-He95"
    type="corners"
    position="right"
    color="purple"
  />
  <VividTyping
    content="车了个车"
    text-white
    style="
      background: -webkit-linear-gradient(120deg, #bd34fe 30%, #41d1ff);
      -webkit-text-fill-color: -webkit-linear-gradient(120deg, #bd34fe 30%, #41d1ff);
      background-clip: text;
    "
    font-bold
    text-center
    text-3xl
    py3
    font-sans
  />
  <div flex="~" justify-center items-center pt2 pb10>
    <div i-carbon-timer />
    {{ countDown }}
  </div>
  <div w-full h-full flex="~ col" items-center>
    <div class="main" relative @click="move" />
    <div class="move-list" />
  </div>
</template>

<style>
#app {
  overflow: hidden;
}
* {
  box-sizing: border-box;
}

.item {
  position: absolute;
  color: #fff;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: left 0.3s, top 0.3s, transform 0.3s;
  border: 1px solid #eee;
  border-radius: 4px;
  cursor: pointer;
}
.item:hover {
  box-shadow: 2px 2px 2px rgb(0 0 0 / 60%);
}
.item:after {
  content: "";
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  transition: background-color 0.2s;
  z-index: 1;
}
.item.disabled:after {
  background-color: rgba(0, 0, 0, 0.7);
  z-index: 0;
}
.item.disabled:hover {
  box-shadow: none;
}
.move-list {
  border: 1px solid #ddd;
  background-color: #eee;
  box-sizing: content-box;
  border-radius: 4px;
}
</style>
