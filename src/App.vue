<template>
  <div class="counter-container">
    <h2>文字数カウントくん</h2>

    <!-- 操作ボタンエリア -->
    <div class="toolbar">
      <button
        class="action-btn"
        @click="pasteText"
        title="クリップボードから貼り付け"
      >
        ペースト
      </button>
      <button
        class="action-btn clear-btn"
        @click="clearText"
        title="テキストをクリア"
      >
        クリア
      </button>
    </div>

    <textarea
      v-model="text"
      ref="textareaRef"
      placeholder="ここにテキストを入力または貼り付けてください..."
      @select="updateSelection"
      @keyup="updateSelection"
      @mouseup="updateSelection"
    ></textarea>

    <!-- オプション設定 -->
    <div class="options">
      <label>
        <input
          type="checkbox"
          v-model="excludeSpaces"
          @change="updateSelection"
        />
        空白（スペース・タブ）を除外してカウント
      </label>
    </div>

    <!-- 統計情報 -->
    <div class="stats">
      <div class="stat-item">
        <span class="label">全体文字数:</span>
        <span class="value">{{ totalCount }}</span>
      </div>
      <div class="stat-item highlight">
        <span class="label">選択中の文字数:</span>
        <span class="value">{{ selectedCount }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

const text = ref("");
const textareaRef = ref(null);
const selectedCount = ref(0);
const excludeSpaces = ref(false);

// 文字数を安全にカウントするヘルパー（絵文字などのサロゲートペア対応）
const getLength = (str) => {
  let target = str;
  if (excludeSpaces.value) {
    target = target.replace(/[\s\u3000]+/g, "");
  }
  return [...target].length;
};

// 全体文字数
const totalCount = computed(() => getLength(text.value));

// 選択範囲の文字数を更新
const updateSelection = () => {
  if (!textareaRef.value) return;

  const start = textareaRef.value.selectionStart;
  const end = textareaRef.value.selectionEnd;

  if (start !== undefined && end !== undefined && start !== end) {
    const selectedText = text.value.substring(start, end);
    selectedCount.value = getLength(selectedText);
  } else {
    selectedCount.value = 0;
  }
};

// クリップボードからテキストを貼り付ける関数
const pasteText = async () => {
  try {
    const clipboardText = await navigator.clipboard.readText();

    if (textareaRef.value) {
      const textarea = textareaRef.value;
      const start = textarea.selectionStart;
      const end = textarea.selectionEnd;

      text.value =
        text.value.substring(0, start) +
        clipboardText +
        text.value.substring(end);

      const newCursorPos = start + clipboardText.length;
      setTimeout(() => {
        textarea.focus();
        textarea.setSelectionRange(newCursorPos, newCursorPos);
        updateSelection();
      }, 0);
    } else {
      text.value += clipboardText;
    }
  } catch (err) {
    console.error("クリップボードの読み取りに失敗しました:", err);
    alert(
      "クリップボードへのアクセスが許可されていないか、対応していないブラウザです。Ctrl+V (Cmd+V) で貼り付けてください。",
    );
  }
};

// テキストをクリアする関数
const clearText = () => {
  text.value = "";
  selectedCount.value = 0;
  if (textareaRef.value) {
    textareaRef.value.focus();
  }
};
</script>

<style src="./App.css"></style>
