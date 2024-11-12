<!--
author:Nico
description:pixijs:a webgl 2d render engine
-->
<script lang="ts" setup>
import { onMounted, ref } from "vue";
import { Application, Assets, Sprite, Container, Graphics, Renderer } from "pixi.js";
const dom = ref();
let app: Application<Renderer>
const init = async () => {
    app = new Application();
    await app.init({
        background: '#1099bb',
        resizeTo: dom.value
    })
    dom.value.appendChild(app.canvas)

    // container
    const container = new Container();
    // container.x = app.canvas.width / 2
    // container.y = app.canvas.height / 2
    app.stage.addChild(container)

    const url = new URL('@/assets/images/hospital.png', import.meta.url);
    const texture = await Assets.load(url.href);
    texture.dynamic = true;
    const sprite = new Sprite(texture)
    sprite.setSize({
        width: app.canvas.width,
        height: app.canvas.height
    })
    sprite.x = app.canvas.width / 2
    sprite.y = app.canvas.height / 2
    sprite.anchor.set(0.5, 0.5)
    container.addChild(sprite)
    // app.stage.addChild(sprite)
    // add click area config  添加可点击区域的配置文件

    areaInfo.map(async item => {
        const coords = generate_coords(item.coords)
        const area = drawArea(coords)
        area.label = item.areaName
        const center = compute_center(coords)
        console.log('center', center);
        // 在此中心插入动图和标识
        const div = document.createElement('div')
        div.style.position = 'absolute'
        div.style.left = center[0] - 34 + 'px';
        div.style.top = center[1] - 40 + 'px';
        const innerHtml =
            `
        <div class="icon-dc">
            <div>${item.areaName}</div>
            <div>
              <div></div>
            </div>
        </div>
      `
        div.innerHTML = innerHtml
        dom.value.appendChild(div)
        // container.addChild(sprite)
        container.addChild(area)
    })
}
// 添加矩形
function drawArea(coords: Array<number>) {
    const graphics = new Graphics()
    graphics.moveTo(coords[0], coords[1])
        .lineTo(coords[2], coords[3])
        .lineTo(coords[4], coords[5])
        .lineTo(coords[6], coords[7])
    // .lineTo(coords[0], coords[1])
    graphics.fill('ff000081');
    graphics.cursor = 'pointer'
    graphics.eventMode = 'static'
    graphics
        .on('pointerdown', onClick, graphics)
        .on('pointerover', over, graphics)
        .on('pointerout', out, graphics)
    return graphics
}
import { ElMessage } from 'element-plus'
function onClick(this: any) {
    console.log("openArea", this);
    ElMessage({
        message: `你点击了${this.label}区域`, type: 'success'
    })
    console.log(0);
}
function over(this: any) {
    this.clear();
    const areaName = this.label;
    const path = areaInfo.filter((item) => {
        return item.areaName === areaName;
    });
    let old_coords = path[0].coords;
    const coords = generate_coords(old_coords);
    this.moveTo(coords[0], coords[1])
        .lineTo(coords[2], coords[3])
        .lineTo(coords[4], coords[5])
        .lineTo(coords[4], coords[5])
        .lineTo(coords[6], coords[7])
        .lineTo(coords[0], coords[1]);
    this.fill("rgba(23,96,205,0.44)");
    this.stroke({
        color: 'rgb(14,96,255)', width: 1
    })
}
function out(this: any) {
    this.clear();
    const areaName = this.label;
    const path = areaInfo.filter((item) => {
        return item.areaName === areaName;
    });
    let old_coords = path[0].coords;
    const coords = generate_coords(old_coords);
    this.moveTo(coords[0], coords[1])
        .lineTo(coords[2], coords[3])
        .lineTo(coords[4], coords[5])
        .lineTo(coords[4], coords[5])
        .lineTo(coords[6], coords[7])
        .lineTo(coords[0], coords[1]);
    this.fill("rgba(255,0,0,0)");
}
// areaInfo
const areaInfo = [
    {
        areaName: "机房01",
        coords: [31.5, 14.8, 38.2, 14.8, 35.8, 44.6, 27.7, 44.6],
    },
    {
        areaName: "机房02",
        coords: [38.2, 14.8, 45.3, 14.4, 44.4, 45.1, 35.8, 44.6],
    },
    {
        areaName: "机房03",
        coords: [45.3, 14.4, 61.8, 14.6, 65, 44, 44, 44.4],
    },
    {
        areaName: "机房04",
        coords: [61.8, 14.6, 79.4, 14.5, 86.8, 44.5, 65, 44],
    },
    {
        areaName: "机房05",
        coords: [66.5, 57.2, 89.7, 56.9, 97.9, 92.1, 70.1, 91.4],
    },
    {
        areaName: "机房06",
        coords: [44.4, 57.4, 66.5, 57.2, 70.1, 91.4, 43.5, 91.1],
    },
];
const generate_coords = (arr: Array<number>) => {
    return arr.map((item, index) => {
        if (index % 2) {
            return (item / 100) * app.renderer.height;
        } else {
            return (item / 100) * app.renderer.width;
        }
    });
};
// 计算中心坐标
const compute_center = (arr: Array<number>) => {
    let len = arr.length / 2
    let x = 0, y = 0;
    arr.map((item, index) => {
        if (index % 2 === 0) {
            x += item
        } else {
            y += item
        }
    })
    return [Math.floor(x / len), Math.floor(y / len)]
}
onMounted(() => {
    init()
})
</script>
<template>
    <div class="container">
        <div class="hospitalCanvas" style="position: relative;" ref="dom">
        </div>
    </div>
</template>

<style scoped lang="scss">
@import url("./common.scss");

.hospitalCanvas {
    height: 100%;
    width: 100%;

}
</style>

<style lang="scss">
.icon-dc {
    // z-index: 100;
    display: grid;
    width: 78px;
    height: 85px;
    grid-template-rows: 28px 1fr;

    >div {
        height: 100%;
        width: 100%;
        display: grid;
        place-items: center;
    }

    >div:first-child {
        background-color: rgba(0, 0, 0, 0.5);
        width: 78px;
        color: white;
        // height: 24px;
    }

    >div:nth-child(2) {
        height: 61px;

        >div {
            height: 100%;
            width: 100%;
            background: url('@/assets/images/dc-areaIcon.png');
            // background-size: 61px 61px;
            background-repeat: no-repeat;
            background-position: center center;
            background-size: contain;
        }

    }
}
</style>