---
title: Vuejs_context menu when right clicking el-table-column
author: B
date: 2023-02-28 16:37:00 +0900
categories: [Vuejs]
---
el-table-column 내에서 우륵 클릭 메뉴를 사용하는 샘플을 찾아봤는데 내가 못찾는건지 아무리 찾아봐도 없어서 그냥 써버리는 contextmenu 예제

처음에는 el-popover로 쉽게 해결될 줄 알았으나.. 이게 웬걸 하필 딱 contextmenu만 동작이 안되고.. (chrome 버전이 안맞는건지 지원이 종료된건지 잘 모르겠다..) 참고 사이트에서 사용하는 popper.js와 vue-click-outside를 사용할 경우에는 미리 사용중이던 node-sass버전을 낮춰야한다고 한다.. 웬만해서 버전 변경하는것에 대해 기피하는 성향이라 귀찮지만 그냥 열심히 굉장한 삽질을 했다.

## PopMenu.vue를 사용하기 위한 Parent Page

```
<template>
    <el-table :data="data">
        <el-table-column>
            <template slot-scopt="scope">
                <PopMenu
                :title="scope.row.title"
                :idx="scope.$index">
                </PopMenu>
            </template>
        </el-table-column>
    </el-table>
</template>
<script>
    import PopMenu from "[PopMenu.vue가 존재하는 위치]";
    export default{
        components: { PopMenu, },
    };
</script>
```

- PopMenu내 Props를 변경하여 전달할 데이터를 추가/수정 할 수 있다.

## PopMenu.vue

```
<template>
  <div @mouseleave="hiddenPop">
    <div @contextmenu.prevent="visiblePop($event, idx)" 
    style="width: 100%; height:100%;">
      <span> {{ title }} </span>
    </div>
    <ul class="pop-menu"
    v-if="visibleIdx === idx"
    :style="popLocation">
      <li class="pop-menu-item"> Menu1 </li>
      <li class="pop-menu-item"> Menu2 </li>
      <li class="pop-menu-item"> Menu3 </li>
    </ul>
  </div>
</template>
<script>
  export default{
    props: {
      title: {
        type: String,
        default: null,
      },
      idx: {
        type: Number,
        default: null,
      },
    },
    data(){
      return{
        visibleIdx : null,
        top: '0px',
        left: '0px',
      };
    },
    computed: {
      popLocation(){
        return{
          top: this.top,
          left: this.left
        };
      },
    },
    methods: {
      visiblePop(evt, idx){
        this.visibleIdx = idx;

        let top = evt.clientY;
        let left = evt.clientX;

        this.top = top + 'px';
        this.left = left + 'px';
      },
      hiddenPop(){
        this.visibleIdx = null;
      },
    },
  };
</script>
<style>
.pop-menu {
  position: fixed;
  z-index: 999;
  margin: 0px;
  padding: 0px;
  overflow: hidden;
  border-radius: 4px;
}

.pop-menu-item {
  display: block;
  position: relative;
  padding: 2px 2px;
  background: #000;
  border-radius: 0;
  color: #FFF;
  text-decoration: none;
  font-size: 13px;
  width: 100%;
  text-align: left;
  cursor: pointer;
  padding:8px;
}

.pop-menu-item:hover,
.pop-menu-item:focus{
  background: #fff;
  color: #000;
  outline: none;
}
</style>
```

- @mouseleave : 마우스가 해당 영역(div)를 떠날 때 사용되는 옵션
- hiddenPop() : 팝업 삭제 합수
- @contextmenu.prevent : 우측 클릭 이벤트 시, 사용되는 옵션
- visiblePop() : 현재 이벤트가 일어난 row의 값을 비교하기 위한 변수 내 값 저장과 이벤트가 발생한 좌표값 지정
- popLocation : visiblePop()에서 저장된 좌표값을 사용하여 ul 태그 출력

다 만들고 정리해보니 딱히 어려울건 없었는데 프론트 쪽에는 익숙하지 않은데다가 vue js는 처음이라 그런지 습득하면서 하느라 시간을 너무 뺏겨버렸다.. 반성하도록

참고 URL : <https://codesandbox.io/s/xplnkv29w4>