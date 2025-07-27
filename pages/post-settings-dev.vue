<template>
  <div class="dev-tools-root">
    <div class="dev-header">
      <h1>公開先 Dev Tools</h1>
      <div class="dev-controls">
        <button @click="resetStyles" class="reset-btn">リセット</button>
        <button @click="applyStyles" class="apply-btn">適用</button>
      </div>
    </div>

    <div class="dev-content">
      <div class="preview-section">
        <h2>プレビュー</h2>
        <div class="preview-container">
          <div class="form-group">
            <label>公開先:</label>
            <select v-model="publishTo" class="publish-select">
              <option value="domestic">国内のみ</option>
              <option value="specific">指定した国</option>
              <option value="public">公開</option>
            </select>
          </div>
          
          <div v-if="publishTo === 'specific'" class="country-select-flex" :style="countrySelectStyles">
            <div class="country-select-item">
              <label>一つ目:</label>
              <select>
                <option>選択</option>
                <option>日本</option>
                <option>アメリカ</option>
                <option>中国</option>
              </select>
            </div>
            <div class="country-select-item">
              <label>二つ目:</label>
              <select>
                <option>選択</option>
                <option>日本</option>
                <option>アメリカ</option>
                <option>中国</option>
              </select>
            </div>
            <div class="country-select-item">
              <label>三つ目:</label>
              <select>
                <option>選択</option>
                <option>日本</option>
                <option>アメリカ</option>
                <option>中国</option>
              </select>
            </div>
          </div>
        </div>
      </div>

      <div class="controls-section">
        <h2>スタイル調整</h2>
        
        <div class="control-group">
          <h3>公開先セレクト</h3>
          <div class="control-item">
            <label>幅:</label>
            <input type="range" v-model="publishSelectWidth" min="100" max="200" step="5">
            <span>{{ publishSelectWidth }}px</span>
          </div>
          <div class="control-item">
            <label>パディング:</label>
            <input type="range" v-model="publishSelectPadding" min="2" max="12" step="1">
            <span>{{ publishSelectPadding }}px</span>
          </div>
        </div>

        <div class="control-group">
          <h3>国選択フレックス</h3>
          <div class="control-item">
            <label>上部マージン:</label>
            <input type="range" v-model="countrySelectMarginTop" min="-50" max="20" step="5">
            <span>{{ countrySelectMarginTop }}px</span>
          </div>
          <div class="control-item">
            <label>下部マージン:</label>
            <input type="range" v-model="countrySelectMarginBottom" min="0" max="60" step="5">
            <span>{{ countrySelectMarginBottom }}px</span>
          </div>
          <div class="control-item">
            <label>ギャップ:</label>
            <input type="range" v-model="countrySelectGap" min="0" max="10" step="1">
            <span>{{ countrySelectGap }}px</span>
          </div>
        </div>

        <div class="control-group">
          <h3>国選択アイテム</h3>
          <div class="control-item">
            <label>ラベル幅:</label>
            <input type="range" v-model="countryItemLabelWidth" min="40" max="80" step="5">
            <span>{{ countryItemLabelWidth }}px</span>
          </div>
          <div class="control-item">
            <label>セレクト幅:</label>
            <input type="range" v-model="countryItemSelectWidth" min="60" max="120" step="5">
            <span>{{ countryItemSelectWidth }}px</span>
          </div>
          <div class="control-item">
            <label>アイテム間ギャップ:</label>
            <input type="range" v-model="countryItemGap" min="0" max="10" step="1">
            <span>{{ countryItemGap }}px</span>
          </div>
        </div>

        <div class="control-group">
          <h3>アクションボタン</h3>
          <div class="control-item">
            <label>上部マージン:</label>
            <input type="range" v-model="actionsMarginTop" min="0" max="80" step="5">
            <span>{{ actionsMarginTop }}px</span>
          </div>
        </div>
      </div>

      <div class="code-section">
        <h2>生成されたCSS</h2>
        <pre class="css-code">{{ generatedCSS }}</pre>
      </div>
    </div>
  </div>
</template>

<script setup>
const publishTo = ref('specific')

// 公開先セレクト
const publishSelectWidth = ref(140)
const publishSelectPadding = ref(8)

// 国選択フレックス
const countrySelectMarginTop = ref(-40)
const countrySelectMarginBottom = ref(0)
const countrySelectGap = ref(2)

// 国選択アイテム
const countryItemLabelWidth = ref(50)
const countryItemSelectWidth = ref(75)
const countryItemGap = ref(1)

// アクションボタン
const actionsMarginTop = ref(15)

const countrySelectStyles = computed(() => ({
  marginTop: `${countrySelectMarginTop.value}px`,
  marginBottom: `${countrySelectMarginBottom.value}px`,
  gap: `${countrySelectGap.value}px`
}))

const generatedCSS = computed(() => {
  return `.publish-select {
  width: ${publishSelectWidth.value}px;
  padding: 5px ${publishSelectPadding.value}px;
  font-size: 1rem;
  border-radius: 0;
  border: 1px solid #bdbdbd;
  background: #fafafa;
  color: #222;
  margin-left: 0;
}

.country-select-flex {
  display: flex;
  gap: ${countrySelectGap.value}px;
  margin-top: ${countrySelectMarginTop.value}px;
  margin-left: 0;
  align-items: center;
  justify-content: flex-start;
  margin-bottom: ${countrySelectMarginBottom.value}px;
}

.country-select-item {
  display: flex;
  align-items: center;
  gap: ${countryItemGap.value}px;
  flex: 1;
  justify-content: flex-start;
  margin-left: 0;
  padding-left: 0;
  margin-right: 0;
  position: relative;
  left: 0;
}

.country-select-item label {
  font-weight: normal;
  font-size: 1rem;
  margin-right: 0px;
  width: ${countryItemLabelWidth.value}px;
  padding-left: 0;
  text-align: left;
  position: relative;
  left: 0;
}

.country-select-item select {
  width: ${countryItemSelectWidth.value}px;
  font-size: 1rem;
  padding: 4px 8px;
  border-radius: 0;
  border: 1px solid #bdbdbd;
  background: #fafafa;
  color: #222;
  min-width: 0;
  margin-left: 0;
  position: relative;
  left: 0;
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 18px;
  margin-top: ${actionsMarginTop.value}px;
}`
})

function resetStyles() {
  publishSelectWidth.value = 140
  publishSelectPadding.value = 8
  countrySelectMarginTop.value = -40
  countrySelectMarginBottom.value = 0
  countrySelectGap.value = 2
  countryItemLabelWidth.value = 50
  countryItemSelectWidth.value = 75
  countryItemGap.value = 1
  actionsMarginTop.value = 15
}

function applyStyles() {
  // メインのpost-settings.vueファイルに適用する処理
  console.log('スタイルを適用しました')
}
</script>

<style scoped>
.dev-tools-root {
  padding: 20px;
  font-family: Arial, sans-serif;
}

.dev-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
  padding-bottom: 15px;
  border-bottom: 2px solid #333;
}

.dev-header h1 {
  margin: 0;
  color: #333;
}

.dev-controls {
  display: flex;
  gap: 10px;
}

.reset-btn, .apply-btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.reset-btn {
  background: #f0f0f0;
  color: #333;
}

.apply-btn {
  background: #007bff;
  color: white;
}

.dev-content {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 30px;
}

.preview-section, .controls-section, .code-section {
  background: #f8f9fa;
  padding: 20px;
  border-radius: 8px;
  border: 1px solid #dee2e6;
}

.preview-section h2, .controls-section h2, .code-section h2 {
  margin-top: 0;
  margin-bottom: 20px;
  color: #333;
  font-size: 18px;
}

.preview-container {
  background: white;
  padding: 20px;
  border-radius: 4px;
  border: 1px solid #ccc;
}

.form-group {
  display: flex;
  align-items: center;
  margin-bottom: 15px;
}

.form-group label {
  width: 80px;
  font-weight: bold;
  margin-right: 10px;
}

.publish-select {
  width: 140px;
  padding: 5px 8px;
  font-size: 1rem;
  border-radius: 0;
  border: 1px solid #bdbdbd;
  background: #fafafa;
  color: #222;
  margin-left: 0;
}

.country-select-flex {
  display: flex;
  gap: 2px;
  margin-top: -40px;
  margin-left: 0;
  align-items: center;
  justify-content: flex-start;
  margin-bottom: 0px;
}

.country-select-item {
  display: flex;
  align-items: center;
  gap: 1px;
  flex: 1;
  justify-content: flex-start;
  margin-left: 0;
  padding-left: 0;
  margin-right: 0;
  position: relative;
  left: 0;
}

.country-select-item label {
  font-weight: normal;
  font-size: 1rem;
  margin-right: 0px;
  width: 50px;
  padding-left: 0;
  text-align: left;
  position: relative;
  left: 0;
}

.country-select-item select {
  width: 75px;
  font-size: 1rem;
  padding: 4px 8px;
  border-radius: 0;
  border: 1px solid #bdbdbd;
  background: #fafafa;
  color: #222;
  min-width: 0;
  margin-left: 0;
  position: relative;
  left: 0;
}

.control-group {
  margin-bottom: 25px;
  padding: 15px;
  background: white;
  border-radius: 4px;
  border: 1px solid #dee2e6;
}

.control-group h3 {
  margin-top: 0;
  margin-bottom: 15px;
  color: #495057;
  font-size: 16px;
}

.control-item {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
  gap: 10px;
}

.control-item label {
  min-width: 120px;
  font-size: 14px;
  color: #495057;
}

.control-item input[type="range"] {
  flex: 1;
  margin: 0 10px;
}

.control-item span {
  min-width: 50px;
  font-size: 14px;
  color: #666;
  text-align: right;
}

.css-code {
  background: #2d3748;
  color: #e2e8f0;
  padding: 15px;
  border-radius: 4px;
  font-family: 'Courier New', monospace;
  font-size: 12px;
  line-height: 1.4;
  overflow-x: auto;
  white-space: pre-wrap;
}
</style> 