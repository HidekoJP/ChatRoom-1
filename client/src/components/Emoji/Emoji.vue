<template>
  <div class="emoji">
    <div class="emoji-line" v-for="(line, index) in emoji" :key="index">
      <div
        class="emoji-item"
        v-for="(item, i) in line"
        :key="i"
        @click="tap(item)"
      >
        {{ item }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      emoji: [
        ["🥵", "😒", "😑", "😐", "😏", "😶", "🙄", "😬"],
        ["🤕", "😵", "😔", "😪", "😴", "🤤", "🤥", "🤨"],
        ["😷", "🤒", "🥴", "🤯", "🤢", "🤐", "🥶", "🤯"],
        ["🤬", "😡", "😤", "😠", "🧐", "🥳", "🤓", "🤠"],
        ["💩", "🤖", "🤡", "👾", "👽", "👻", "👺", "👹"],
        ["😵‍💫", "😚", "😙", "😗", "😘", "🤩", "😍", "🥰"],
        ["😴", "🤤", "😝", "🤪", "😜", "😛", "🤑", "😋"],
        ["🤕", "😵", "😔", "😪", "😴", "🤤", "🤥", "🤨"],
        ["😷", "🤒", "🥴", "🤯", "🤢", "🤐", "🥶", "🤯"],
        ["🤬", "😡", "😤", "😠", "🧐", "🥳", "🤓", "🤠"],
        ["💩", "🤖", "🤡", "👾", "👽", "👻", "👺", "👹"],
        ["☺", "😚", "😙", "😗", "😘", "🤩", "😍", "🥰"],
        ["😴", "🤤", "😝", "🤪", "😜", "", "", ""],
      ],
    };
  },
  methods: {
    // 获取表情并发送
    tap(item) {
      this.$emit("sendEmoji", item);
    },
  },
};
</script>

<style scoped>
.emoji {
  width: 100%;
  height: 230px;
  padding: 10px;
  box-sizing: border-box;
  overflow: hidden;
  overflow-y: auto;
}
.emoji-line {
  display: flex;
  text-align: center;
  padding-top: 12px;
  font-size: 24px;
  line-height: 35px;
}
.emoji-item {
  flex: 1;
}
</style>