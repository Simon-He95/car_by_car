<script setup lang="ts">
import { onMounted } from "@vue/runtime-core";
import { addEventListener, findElement, removeElement } from "simon-js-tool";
import gitForkVue from "@simon_he/git-fork-vue";
// base
const data = [
  {
    src: new URL("../../public/1.png", import.meta.url).href,
    color: "#ff1100",
  },
  {
    src: new URL("../../public/2.png", import.meta.url).href,
    color: "#ff8800",
  },
  { src: new URL("../../public/3.png", import.meta.url).href, color: "green" },
  {
    src: new URL("../../public/4.png", import.meta.url).href,
    color: "#779922",
  },
  { src: new URL("../../public/5.png", import.meta.url).href, color: "blue" },
  {
    src: new URL("../../public/6.png", import.meta.url).href,
    color: "#335577",
  },
];
// card size
const size = 35;

const rows = 10;
const cols = 10;
// eliminate the number of cards
const eliminateCount = 3;
// group
const group = 6;
// layer
const layerCount = 6;
const cellHtml: string[] = [];

let moveList: HTMLElement;
let main: HTMLElement;
let gameOver = false;
const renderData = Array.from({ length: eliminateCount * group }, (_, i) =>
  data.map((v) => ({ ...v }))
)
  .flat()
  .sort(() => Math.random() - 0.5);
for (let ly = layerCount - 1; ly >= 0; ly--) {
  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      const pyStep = (ly + 1) % 2 === 0 ? size / 2 : 0;
      const item = Math.random() > 0.7 && renderData.pop();
      if (item) {
        cellHtml.push(
          `<div class="item" id="m${ly}-${i}-${j}" style="width:${size}px;height:${size}px;left:${
            size * j + pyStep
          }px;top:${size * i + pyStep}px;background-color:${
            item.color
          }"><img src="${item.src}"/></div>`
        );
      }
    }
  }
}

const checkDisabled = () => {
  findElement(".item", true)?.forEach((v, i) => {
    const arr = v.id.substring(1).split("-").map(Number);
    const isPy = (arr[0] + 1) % 2 === 0;
    for (let i = arr[0] + 1; i <= layerCount - 1; i++) {
      const isPyB = (i + 1) % 2 === 0;
      if (isPy === isPyB) {
        const el = findElement(`#m${i}-${arr[1]}-${arr[2]}`, main);
        if (el) v.classList.add("disabled");
      } else if (isPy && !isPyB) {
        const result = [
          `${i}-${arr[1]}-${arr[2]}`,
          `${i}-${arr[1]}-${arr[2] + 1}`,
          `${i}-${arr[1] + 1}-${arr[2]}`,
          `${i}-${arr[1] + 1}-${arr[2] + 1}`,
        ].every((k) => !findElement(`#m${k}`, main));
        if (!result) v.classList.add("disabled");
        else v.classList.remove("disabled");
      } else if (!isPy && isPyB) {
        const result = [
          `${i}-${arr[1]}-${arr[2]}`,
          `${i}-${arr[1]}-${arr[2] - 1}`,
          `${i}-${arr[1] - 1}-${arr[2]}`,
          `${i}-${arr[1] - 1}-${arr[2] - 1}`,
        ].every((k) => !findElement(`#m${k}`, main));
        if (!result) v.classList.add("disabled");
        else v.classList.remove("disabled");
      }
    }
  });
};
onMounted(() => {
  main = findElement(".main")!;
  moveList = findElement(".move-list")!;
  main.innerHTML = cellHtml.reverse().join("");
  main.style.height = `${size * rows + size * 2}px`;
  main.style.width = `${size * cols}px`;
  moveList.style.height = `${size}px`;
  moveList.style.width = `${size * group}px`;
  checkDisabled();
});
const move = (e: MouseEvent) => {
  if (gameOver) return;
  const target = e.target as HTMLElement;
  if (!target.className.includes("item")) return;
  const top = moveList.offsetTop - main.offsetTop;
  if (target.className.includes("disabled")) return;

  const left = (main.offsetWidth - moveList.offsetWidth) / 2;
  const len = moveList.children.length;
  target.style.top = `${top}px`;
  target.style.left = `${len * size + left}px`;
  let count = 0;
  const eliminateCheck = () => {
    const moveListChildren = Array.from(moveList.children).filter(
      (v) => v.innerHTML === target.innerHTML
    ) as HTMLElement[];
    const resetChildren = main.children;
    if (moveListChildren.length === 3) {
      return moveListChildren.forEach((v) => {
        v.style.transform = "scale(0)";
        setTimeout(() => {
          removeElement(v);
        }, 300);
      });
    }
    if (moveList.children.length === group) {
      gameOver = true;
      return alert("You lose!");
    }
    if (resetChildren.length === 0) {
      gameOver = true;
      return alert("You win!");
    }
    checkDisabled();
    setTimeout(checkDisabled);
  };
  const stop = addEventListener(target, "transitionend", () => {
    count++;
    if (count === 2) {
      stop();
      target.style.top = "0px";
      target.style.left = "0px";
      target.style.position = "relative";
      target.style.display = "inline-flex";
      const newTarget = target.cloneNode(true) as HTMLElement;
      moveList.appendChild(newTarget);
      eliminateCheck();
      removeElement(target);
    }
  });
};
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
      -webkit-text-fill-color: -webkit-linear-gradient(
        120deg,
        #bd34fe 30%,
        #41d1ff
      );
      background-clip: text;
    "
    font-bold
    text-center
    text-3xl
    py3
    font-sans
  />
  <div w-full h-full flex="~ col" items-center mt18>
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
