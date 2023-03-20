<style lang="scss" scoped>
.stage {
  max-width: 750px;
  margin: 0 auto;
  .n-button {
    font-size: 40px;
  }
  .top-bar {
    display: flex;
    justify-content: space-between;
    padding: 10px;
    font-size: 20px;
  }
  .numbers {
    display: flex;
    flex-wrap: wrap;
    .button-box {
      width: 50%;
      padding: 10px;
      .n-button,
      .deleted-box {
        width: 100%;
        height: 100px;
      }
    }
    .deleted-box {
      background: #eee;
    }
  }
  .actions {
    margin-top: 20px;
    display: flex;
    justify-content: space-around;
    .n-button {
      width: 60px;
      height: 60px;
      line-height: 60px;
    }
  }

  .reset-action {
    margin-top: 20px;
    margin: 20px;
    .n-button {
      height: 60px;
    }
  }
}
</style>
<template lang="pug">
.stage
  .top-bar
    .time 剩余 {{ state.time }} s
    .count 答对 {{ state.count }} 道

  .numbers
    .button-box(v-for="number in state.numbers")
      NButton(
        v-if="!number.deleted",
        strong,
        secondary,
        :type="state.cache.includes(number) ? 'info' : 'primary'",
        @click="click(number)"
      ) {{ number.value }}
      .deleted-box(v-else)

  .actions
    NButton(
      :type="state.cache.includes(action) ? 'info' : 'primary'",
      v-for="action in state.actions",
      @click="click(action)"
    ) {{ action.value }}

  .reset-action
    NButton(strong, secondary, type="error", @click="reset", block) 复原
</template>
<script setup>
import { reactive } from "vue";
import questions from "./questions";
import { NButton, useDialog } from "naive-ui";
import clonedeep from "lodash.clonedeep";

const getNextQuestion = () => {
  if (!questions.length) {
    // alert("你真棒，题都做完了~");
    return;
  }
  const index = Math.floor(Math.random() * questions.length);
  state.numbers = questions[index].slice(0, 4).map((number) => {
    return {
      type: "number",
      value: number,
      checked: false,
      deleted: false,
    };
  });
  state.numbers2 = clonedeep(state.numbers);
  questions.splice(index, 1);
};
const state = reactive({
  numbers: [],
  numbers2: [],
  actions: ["+", "-", "*", "/"].map((action) => ({
    type: "action",
    value: action,
    checked: false,
  })),
  deleted: 0,
  cache: [],
  count: 0,
  time: 300,
  timer: null,
});

const click = (block) => {
  const type = block.type;

  const cacheLength = state.cache.length;
  if (type === "action") {
    // 第一个点的是操作则无效
    if (cacheLength === 0) {
      return;
    }
    // 如果是第二个点的就加进来
    if (cacheLength === 1) {
      state.cache.push(block);
      return;
    }
    // 如果是第三个就替换第二个
    if (cacheLength === 2) {
      state.cache[1] = block;
      return;
    }
    return;
  }

  if (cacheLength === 0) {
    state.cache.push(block);
    return;
  }
  if (cacheLength === 1) {
    state.cache[0] = block;
    return;
  }
  if (cacheLength === 2) {
    state.cache.push(block);
    // 计算
    const result = eval(
      `${state.cache[0].value} ${state.cache[1].value} ${state.cache[2].value}`
    );
    state.cache[0].deleted = true;
    state.cache[2].value = result;
    state.cache = [state.cache[2]];
    state.deleted++;

    if (state.deleted >= 3 && result === 24) {
      doNext();
    }
  }
};
const doNext = () => {
  getNextQuestion();
  state.cache = [];
  state.count++;
  state.deleted = 0;
};

const reset = () => {
  state.cache = [];
  state.numbers = clonedeep(state.numbers2);
};
const countdown = () => {
  state.timer = setInterval(() => {
    state.time--;
    if (state.time === 0) {
      clearInterval(state.timer);

      dialog.success({
        title: `你完成了${state.count}道！`,
        content: state.count >= 10 ? "神，大神！" : "继续努力吧!",
        positiveText: "点击重新开始游戏",
        closable: false,
        closeOnEsc: false,
        maskClosable: false,
        onPositiveClick: () => {
          doNext();
          state.count = 0;
          state.time = 300;
          countdown();
        },
      });
    }
  }, 1000);
};

const dialog = useDialog();
dialog.info({
  title: "24点游戏",
  content: "在5分钟的时间内，试一试能做对几道吧",
  positiveText: "点击开始",
  closable: false,
  closeOnEsc: false,
  maskClosable: false,
  onPositiveClick: () => {
    countdown();
  },
});

getNextQuestion();
</script>
