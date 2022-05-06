<template>
  <div>
    <div>
      切换布局
      <el-switch
        v-model="statusForShow"
        class="ml-2"
        active-color="#13ce66"
        inactive-color="#ff4949"
      />
    </div>

    <div :class="{drag:dragging}">
      <div class="layout-container" :style="{'flex-direction':statusForShow?'column':'row'}">
        <div v-for="(item, key) in mainData" :key="key" :class="key">
          <Draggable
            v-bind="dragOptions"
            class="list-group"
            :list="item"
            :item-key="key"
            animation="500"
            @start="onStart"
            @end="onEnd"
          >
            <template #item="{element,index}">
              <div class="content-box" :style="{flex:statusForShow?element.flexStyle || 1:'none'}">
                <div style="display: flex;flex-direction: row">
                  <div class="drag-handle" style="flex: 7">{{ element.title }}</div>
                  <div style="flex: 1" @click="changeModel($event,key,element,)">按钮</div>
                </div>
                <div class="component-box">
                  <component :is="element.name" v-bind="element.data" :ref="`dragRef${index}`" :key="element.name"/>
                </div>
              </div>
            </template>
          </Draggable>
        </div>
      </div>
    </div>
  </div>
</template>
<script lang="ts">
import {
  Test1,
  Test2,
  Test3,
  Test4,
  Test5,
  Test6,
  Test7,
  Test8
} from './components/DragComponents'

export default {
  components: {
    Test1,
    Test2,
    Test3,
    Test4,
    Test5,
    Test6,
    Test7,
    Test8
  }
}
</script>
<script setup lang="ts">
/* eslint-disable */
import Draggable from 'vuedraggable'
import { computed, ref, reactive, onMounted, watchEffect } from 'vue'

const dragging = ref(false)
const componentList = reactive([
  {
    name: 'Test1',
    title: '测试1',
    id: '1'
  },
  {
    name: 'Test2',
    title: '测试2',
    id: '2'
  },
  {
    name: 'Test3',
    title: '测试3',
    id: '3'
  },
  {
    name: 'Test4',
    title: '测试4',
    id: '4'
  },
  {
    name: 'Test5',
    title: '测试5',
    id: '5'
  },
  {
    name: 'Test6',
    title: '测试6',
    id: '6'
  },
  {
    name: 'Test7',
    title: '测试7',
    id: '7'
  },
  {
    name: 'Test8',
    title: '测试8',
    id: '8'
  }
])
const layout = reactive({
  'rowsLayout-left': ['1', '2', '3'],
  'rowsLayout-middle': ['4', '5', '6'],
  'rowsLayout-right': ['7', '8']
})
const changeModel = (e: { preventDefault: () => void; stopPropagation: () => void; }, prop: string | number, ele: { name: any; }) => {
  e.preventDefault()
  e.stopPropagation()
  mainData[prop].forEach((item: { name: any; flexStyle: number; }) => {
    if (item.name === ele.name) {
      item.flexStyle = 2
    }
  })
}

let mainData = reactive<any>({})
const dragOptions = computed(() => {
  return {
    animation: 500,
    handle: '.drag-handle',
    group: 'description',
    ghostClass: 'ghost',
    chosenClass: 'sortable',
    forceFallback: true
  }
})
onMounted(() => {
  getLayout()
})

const onStart = () => {
  dragging.value = true
}
const onEnd = () => {
  dragging.value = false
  setLayout()
}

const getLayout = () => {
  let myLayout = JSON.parse(window.localStorage.getItem('dragLayout') as string)
  if (!myLayout || Object.keys(myLayout).length === 0 || !includes(myLayout)) {
    myLayout = layout
  }
  const newLayout: any = {}
  for (const side in myLayout) {
    if (myLayout[side].length > 0) {
      newLayout[side] = myLayout[side].map((i: string) => {
        return componentList.find(c => c.id === i)
      })
    }
  }
  mainData = Object.assign(mainData, newLayout)
}

const setLayout = () => {
  const res: any = {}
  for (const side in mainData) {
    res[side] = mainData[side].map((i: { id: any; }) => i.id)
  }
  window.localStorage.setItem('dragLayout', JSON.stringify(res))
}
const includes = (prop: { [s: string]: unknown; } | ArrayLike<unknown>) => {
  const layoutArr = ['rowsLayout', 'columnsLayout', 'gridLayout']
  const nowStr = Object.entries(prop)[0][0]
  const index = nowStr.indexOf('-')
  return index > -1 ? layoutArr.includes(nowStr.slice(0, index)) : false
}
const changeLayout = (layouts: string) => {
  const newLayout: any = {}
  for (const key in mainData) {
    const aliasName = key.slice(key.indexOf('-'))
    const layoutName = layouts + aliasName
    newLayout[layoutName] = mainData[key]
    delete mainData[key]
  }

  mainData = Object.assign(mainData, newLayout)
}
const statusForShow = ref(true)
watchEffect(() => {
  statusForShow.value ? changeLayout('rowsLayout') : changeLayout('columnsLayout')
})
</script>
<style lang="scss" scoped>
.layout-container {
  height: 100%;
  display: flex;
  // rowsLayout
  .rowsLayout-left {
    //border: 1px dashed #ccc;
    max-height: 360px;
    min-height: 100px;

    .list-group {
      display: flex;
      flex-direction: row;
      flex-wrap: wrap;
      //justify-content: space-around;

      .content-box {
        height: 240px;
        margin: 5px;

        &:nth-child(1) {
          flex: 1;
        }

        &:nth-child(2) {
          flex: 2;
        }

        &:nth-child(3) {
          flex: 1;
        }

        .component-box {
          height: calc(100% - 60px);
        }
      }
    }
  }

  .rowsLayout-middle {
    max-height: 360px;
    min-height: 100px;
    //border: 1px dashed #ccc;

    .list-group {
      display: flex;
      flex-direction: row;
      flex-wrap: wrap;
      //justify-content: space-around;

      .content-box {
        height: 280px;
        margin: 5px;

        &:nth-child(1) {
          flex: 2;
        }

        &:nth-child(2) {
          flex: 1;
        }

        &:nth-child(3) {
          flex: 2;
        }

        .component-box {
          height: calc(100% - 60px);
        }
      }
    }

  }

  .rowsLayout-right {
    max-height: 400px;
    //border: 1px dashed #ccc;
    min-height: 100px;

    .list-group {
      display: flex;
      flex-direction: row;
      flex-wrap: wrap;
      justify-content: space-around;

      .content-box {
        height: 300px;
        margin: 5px;
        flex: 1;

        .component-box {
          height: calc(100% - 60px);
        }
      }
    }
  }

  // columnsLayout
  .columnsLayout-left {
    //border: 1px dashed #ccc;
    min-height: 600px;
    flex: 1;

    .list-group {
      display: flex;
      min-height: 600px;
      flex-direction: column;
      flex-wrap: wrap;
      //justify-content: space-around;

      .content-box {
        height: 240px;
        margin: 5px;

        .component-box {
          height: calc(100% - 60px);
        }
      }
    }
  }

  .columnsLayout-middle {
    min-height: 600px;
    //border: 1px dashed #ccc;
    flex: 1;

    .list-group {
      display: flex;
      min-height: 600px;
      flex-direction: column;
      flex-wrap: wrap;

      .content-box {
        height: 280px;
        margin: 5px;

        .component-box {
          height: calc(100% - 60px);
        }
      }
    }

  }

  .columnsLayout-right {
    min-height: 600px;
    flex: 1;

    .list-group {
      display: flex;
      min-height: 600px;
      flex-direction: column;
      flex-wrap: wrap;

      .content-box {
        height: 300px;
        margin: 5px;

        .component-box {
          height: calc(100% - 60px);
        }
      }
    }
  }

  .list-group-item {
    flex: 1;
    margin-bottom: 20px;
    border-radius: 6px;
    overflow: hidden;
    background: #fff;
  }

  .content-box {
    box-shadow: 1px 1px 10px #ccc;
  }

  .component-box {
    padding: 10px;
  }

  .drag-handle {
    cursor: move;
    height: 40px;
    line-height: 40px;
    color: #000;
    font-weight: 700;
    font-size: 16px;
    padding: 0 20px;
  }
}

.drag {
  .component-box {
    display: none;
  }
}

.list-enter-active {
  transition: all .3s linear;

}

.list-enter,
.list-leave-to {
  opacity: .5;
}

.sortable {
  .component-box {
    display: none;
    height: 0;
  }
}

.list-group {
  > span {
    display: block;
    min-height: 20px;
  }
}

.ghost {
  .drag-handle {
    background: #fff;
    border-right: 0;
    box-shadow: 1px 1px 4px #ccc;
  }
}
</style>
