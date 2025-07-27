<template>
  <div class="post-settings-root">
    <div class="lang-switcher panel-width">
      <button :class="{active: lang==='ja'}" @click="lang='ja'">日本語</button>
      <button :class="{active: lang==='en'}" @click="lang='en'">English</button>
    </div>
    <div class="window-bar panel-width bar-balance">
      <div class="window-bar-btns">
        <span class="bar-btn red"></span>
        <span class="bar-btn yellow"></span>
        <span class="bar-btn green"></span>
      </div>
      <div class="window-bar-title">{{ t('title') }}</div>
    </div>
    <div class="post-settings-panel panel-width">
      <div class="post-settings-left">
        <div class="image-upload-placeholder"></div>
      </div>
      <div class="post-settings-right">
        <form @submit.prevent="onUpload">
          <div class="form-group">
            <label>{{ t('name') }}<span class="required-asterisk">*</span>：</label>
            <input v-model="form.name" type="text" :placeholder="t('namePh')" :class="{'input-error': errors.name}" />
          </div>
          <div class="form-group">
            <label>{{ t('headline') }}<span class="required-asterisk">*</span>：</label>
            <input v-model="form.headline" type="text" :placeholder="t('headlinePh')" :class="{'input-error': errors.headline}" />
          </div>
          <div class="form-group">
            <label>{{ t('subHeadline') }}<span class="required-asterisk">*</span>：</label>
            <input v-model="form.subHeadline" type="text" :placeholder="t('subHeadlinePh')" :class="{'input-error': errors.subHeadline}" />
          </div>
          <div class="form-group">
            <label>{{ t('hashtags') }}<span class="required-asterisk">*</span>：</label>
            <input v-model="form.hashtags" type="text" :placeholder="t('hashtagsPh')" :class="{'input-error': errors.hashtags}" />
          </div>
          <div class="form-group">
            <label>{{ t('references') }}：</label>
            <input v-model="form.references" type="text" :placeholder="t('referencesPh')" />
          </div>
          <div class="form-group">
            <label>{{ t('aiReplyPattern') }}：</label>
            <input v-model="form.aiReplyPattern" type="text" :placeholder="t('aiReplyPatternPh')" />
          </div>
          <div class="form-row select-row">
            <div class="form-group small">
              <label>{{ t('aiSummary') }}：</label>
              <select v-model="form.aiSummary">
                <option value="on">{{ t('on') }}</option>
                <option value="off">{{ t('off') }}</option>
              </select>
            </div>
            <div class="form-group small">
              <label>{{ t('likes') }}：</label>
              <select v-model="form.likes">
                <option value="show">{{ t('show') }}</option>
                <option value="hide">{{ t('hide') }}</option>
              </select>
            </div>
            <div class="form-group small">
              <label>{{ t('saves') }}：</label>
              <select v-model="form.saves">
                <option value="show">{{ t('show') }}</option>
                <option value="hide">{{ t('hide') }}</option>
              </select>
            </div>
            <div class="form-group small">
              <label>{{ t('comments') }}：</label>
              <select v-model="form.comments">
                <option value="on">{{ t('on') }}</option>
                <option value="off">{{ t('off') }}</option>
              </select>
            </div>
            <div class="form-group small">
              <label>{{ t('payment') }}：</label>
              <select v-model="form.payment">
                <option value="on">{{ t('on') }}</option>
                <option value="off">{{ t('off') }}</option>
              </select>
            </div>
          </div>
          <!-- 公開先セクションを独立したdivに分離 -->
          <div class="publish-section">
            <div class="publish-main">
              <label>{{ t('publishTo') }}：</label>
              <select v-model="form.publishTo" class="publish-select" @change="onPublishToChange">
                <option value="domestic">{{ t('domestic') }}</option>
                <option value="specific">{{ t('specific') }}</option>
                <option value="public">{{ t('public') }}</option>
              </select>
            </div>

            <div v-if="form.publishTo === 'specific'" class="publish-countries">
              <div class="country-select-item">
                <label>{{ t('first') }}：</label>
                <select v-model="form.selectedCountries[0]" :size="1">
                  <option value="">{{ t('select') }}</option>
                  <option v-for="country in countryList" :key="country.code" :value="country.code" :disabled="form.selectedCountries.includes(country.code) && form.selectedCountries[0] !== country.code">
                    {{ country.name }}
                  </option>
                </select>
              </div>
              <div class="country-select-item">
                <label>{{ t('second') }}：</label>
                <select v-model="form.selectedCountries[1]" :size="1">
                  <option value="">{{ t('select') }}</option>
                  <option v-for="country in countryList" :key="country.code" :value="country.code" :disabled="form.selectedCountries.includes(country.code) && form.selectedCountries[1] !== country.code">
                    {{ country.name }}
                  </option>
                </select>
              </div>
              <div class="country-select-item">
                <label>{{ t('third') }}：</label>
                <select v-model="form.selectedCountries[2]" :size="1">
                  <option value="">{{ t('select') }}</option>
                  <option v-for="country in countryList" :key="country.code" :value="country.code" :disabled="form.selectedCountries.includes(country.code) && form.selectedCountries[2] !== country.code">
                    {{ country.name }}
                  </option>
                </select>
              </div>
            </div>
          </div>
          <div v-if="errorMsg" class="error-msg">{{ errorMsg }}</div>
          <div class="form-actions">
            <button type="button" class="close-btn" @click="onClose">{{ t('close') }}</button>
            <button type="button" class="export-btn" @click="onExport">{{ t('export') }}</button>
            <button type="submit" class="upload-btn" :disabled="!canUpload">{{ t('upload') }}</button>
          </div>
        </form>
        <div v-if="showPublishTime" class="publish-time-msg">
          {{ t('publishMsg', { time: fakeTime }) }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

const lang = ref('ja')
const dict = {
  ja: {
    title: '投稿設定',
    name: '名前', namePh: 'プロジェクト名を入力 ...',
    headline: '見出し', headlinePh: 'ここが注目のキーとなります！',
    subHeadline: '～小見出し～', subHeadlinePh: 'ここもこだわりましょう！',
    hashtags: 'ハッシュタグ', hashtagsPh: '見てもらいたい人に見てもらいやすくしましょう！',
    references: '参考文献、その他リンク', referencesPh: '信憑性を高めるためのとても重要な操作です ...',
    aiReplyPattern: 'AIによる返答パターンの作成', aiReplyPatternPh: 'ユーザからきそうな質問に予め回答 ...',
    aiSummary: 'AI要約', likes: 'いいね数', saves: '保存数', comments: 'コメント機能', payment: '送金機能',
    on: 'オン', off: 'オフ', show: '表示', hide: '非表示',
    publishTo: '公開先', domestic: '国内のみ', specific: '指定した国', public: 'パブリック',
    first: '一つ目', second: '二つ目', third: '三つ目', select: '選択',
    close: '閉じる', export: '書き出し', upload: 'アップロード',
    publishMsg: ({ time }: { time: string }) => `あなたの記事は${time}に公開されます！`,
    errorRequired: '必須項目を入力してください',
    errorCountry: '国を1つ以上選択してください',
    errorCountryDup: '同じ国は重複して選択できません',
    errorHashtag: 'ハッシュタグを1つ以上入力してください',
  },
  en: {
    title: 'Post Settings',
    name: 'Name', namePh: 'Enter project name ...',
    headline: 'Headline', headlinePh: 'This will be the key point!',
    subHeadline: 'Subheadline', subHeadlinePh: 'Let’s add more details here!',
    hashtags: 'Hashtags', hashtagsPh: 'Make it easy for people to find your post!',
    references: 'References / Other Links', referencesPh: 'Adding references increases credibility ...',
    aiReplyPattern: 'AI Reply Pattern', aiReplyPatternPh: 'Pre-answer possible user questions ...',
    aiSummary: 'AI Summary', likes: 'Likes', saves: 'Saves', comments: 'Comments', payment: 'Payment',
    on: 'On', off: 'Off', show: 'Show', hide: 'Hide',
    publishTo: 'Visibility', domestic: 'Domestic Only', specific: 'Specific Countries', public: 'Public',
    first: 'First', second: 'Second', third: 'Third', select: 'Select',
    close: 'Close', export: 'Export', upload: 'Upload',
    publishMsg: ({ time }: { time: string }) => `Your post will be published at ${time}!`,
    errorRequired: 'Please fill in all required fields',
    errorCountry: 'Please select at least one country',
    errorCountryDup: 'You cannot select the same country more than once',
    errorHashtag: 'Please enter at least one hashtag',
  }
} as const

type LangKey = keyof typeof dict
const t = (key: keyof typeof dict['ja'], params?: { time?: string }) => {
  const value = dict[lang.value as LangKey][key]
  if (typeof value === 'function') {
    return (value as any)({ time: params?.time || '' })
  }
  return value
}

const countryListJa = [
  { code: 'JP', name: '日本' }, { code: 'US', name: 'アメリカ合衆国' }, { code: 'GB', name: 'イギリス' }, { code: 'CA', name: 'カナダ' }, { code: 'AU', name: 'オーストラリア' },
  { code: 'DE', name: 'ドイツ' }, { code: 'FR', name: 'フランス' }, { code: 'IT', name: 'イタリア' }, { code: 'ES', name: 'スペイン' }, { code: 'CN', name: '中国' },
  { code: 'KR', name: '韓国' }, { code: 'RU', name: 'ロシア' }, { code: 'IN', name: 'インド' }, { code: 'BR', name: 'ブラジル' }, { code: 'MX', name: 'メキシコ' },
  { code: 'ID', name: 'インドネシア' }, { code: 'TR', name: 'トルコ' }, { code: 'SA', name: 'サウジアラビア' }, { code: 'ZA', name: '南アフリカ' }, { code: 'EG', name: 'エジプト' },
  { code: 'AR', name: 'アルゼンチン' }, { code: 'UA', name: 'ウクライナ' }, { code: 'SE', name: 'スウェーデン' }, { code: 'NO', name: 'ノルウェー' }, { code: 'DK', name: 'デンマーク' },
  { code: 'FI', name: 'フィンランド' }, { code: 'PL', name: 'ポーランド' }, { code: 'NL', name: 'オランダ' }, { code: 'BE', name: 'ベルギー' }, { code: 'CH', name: 'スイス' },
  { code: 'AT', name: 'オーストリア' }, { code: 'GR', name: 'ギリシャ' }, { code: 'CZ', name: 'チェコ' }, { code: 'HU', name: 'ハンガリー' }, { code: 'PT', name: 'ポルトガル' },
  { code: 'IE', name: 'アイルランド' }, { code: 'NZ', name: 'ニュージーランド' }, { code: 'SG', name: 'シンガポール' }, { code: 'TH', name: 'タイ' }, { code: 'MY', name: 'マレーシア' },
  { code: 'PH', name: 'フィリピン' }, { code: 'VN', name: 'ベトナム' }, { code: 'IL', name: 'イスラエル' }, { code: 'AE', name: 'アラブ首長国連邦' }, { code: 'QA', name: 'カタール' },
  { code: 'KW', name: 'クウェート' }, { code: 'CL', name: 'チリ' }, { code: 'CO', name: 'コロンビア' }, { code: 'PE', name: 'ペルー' }, { code: 'VE', name: 'ベネズエラ' },
  { code: 'PK', name: 'パキスタン' }, { code: 'BD', name: 'バングラデシュ' }, { code: 'NG', name: 'ナイジェリア' }, { code: 'KE', name: 'ケニア' }, { code: 'MA', name: 'モロッコ' },
  { code: 'TN', name: 'チュニジア' }, { code: 'DZ', name: 'アルジェリア' }, { code: 'RO', name: 'ルーマニア' }, { code: 'BG', name: 'ブルガリア' }, { code: 'HR', name: 'クロアチア' },
  { code: 'SK', name: 'スロバキア' }, { code: 'SI', name: 'スロベニア' }, { code: 'RS', name: 'セルビア' }, { code: 'BA', name: 'ボスニア・ヘルツェゴビナ' }, { code: 'MK', name: 'マケドニア' },
  { code: 'ME', name: 'モンテネグロ' }, { code: 'AL', name: 'アルバニア' }, { code: 'LT', name: 'リトアニア' }, { code: 'LV', name: 'ラトビア' }, { code: 'EE', name: 'エストニア' },
  { code: 'BY', name: 'ベラルーシ' }, { code: 'GE', name: 'ジョージア' }, { code: 'AM', name: 'アルメニア' }, { code: 'AZ', name: 'アゼルバイジャン' }, { code: 'KZ', name: 'カザフスタン' },
  { code: 'UZ', name: 'ウズベキスタン' }, { code: 'KG', name: 'キルギス' }, { code: 'TJ', name: 'タジキスタン' }, { code: 'TM', name: 'トルクメニスタン' }, { code: 'MD', name: 'モルドバ' },
  { code: 'LU', name: 'ルクセンブルク' }, { code: 'LI', name: 'リヒテンシュタイン' }, { code: 'MC', name: 'モナコ' }, { code: 'SM', name: 'サンマリノ' }, { code: 'VA', name: 'バチカン市国' },
  { code: 'AD', name: 'アンドラ' }, { code: 'MT', name: 'マルタ' }, { code: 'CY', name: 'キプロス' }, { code: 'IS', name: 'アイスランド' }, { code: 'PY', name: 'パラグアイ' },
  { code: 'UY', name: 'ウルグアイ' }, { code: 'BO', name: 'ボリビア' }, { code: 'EC', name: 'エクアドル' }, { code: 'GT', name: 'グアテマラ' }, { code: 'HN', name: 'ホンジュラス' },
  { code: 'SV', name: 'エルサルバドル' }, { code: 'NI', name: 'ニカラグア' }, { code: 'CR', name: 'コスタリカ' }, { code: 'PA', name: 'パナマ' }, { code: 'CU', name: 'キューバ' },
  { code: 'DO', name: 'ドミニカ共和国' }, { code: 'HT', name: 'ハイチ' }, { code: 'JM', name: 'ジャマイカ' }, { code: 'BS', name: 'バハマ' }, { code: 'BB', name: 'バルバドス' },
  { code: 'TT', name: 'トリニダード・トバゴ' }, { code: 'GD', name: 'グレナダ' }, { code: 'LC', name: 'セントルシア' }, { code: 'VC', name: 'セントビンセント・グレナディーン' }, { code: 'AG', name: 'アンティグア・バーブーダ' },
  { code: 'KN', name: 'セントクリストファー・ネイビス' }, { code: 'BZ', name: 'ベリーズ' }, { code: 'GY', name: 'ガイアナ' }, { code: 'SR', name: 'スリナム' }, { code: 'FJ', name: 'フィジー' },
  { code: 'PG', name: 'パプアニューギニア' }, { code: 'SB', name: 'ソロモン諸島' }, { code: 'VU', name: 'バヌアツ' }, { code: 'WS', name: 'サモア' }, { code: 'TO', name: 'トンガ' },
  { code: 'TV', name: 'ツバル' }, { code: 'NR', name: 'ナウル' }, { code: 'PW', name: 'パラオ' }, { code: 'FM', name: 'ミクロネシア連邦' }, { code: 'MH', name: 'マーシャル諸島' },
  { code: 'TL', name: '東ティモール' }, { code: 'KH', name: 'カンボジア' }, { code: 'LA', name: 'ラオス' }, { code: 'MM', name: 'ミャンマー' }, { code: 'NP', name: 'ネパール' },
  { code: 'LK', name: 'スリランカ' }, { code: 'MV', name: 'モルディブ' }, { code: 'BT', name: 'ブータン' }, { code: 'MN', name: 'モンゴル' }, { code: 'AF', name: 'アフガニスタン' },
  { code: 'IR', name: 'イラン' }, { code: 'IQ', name: 'イラク' }, { code: 'SY', name: 'シリア' }, { code: 'JO', name: 'ヨルダン' }, { code: 'LB', name: 'レバノン' },
  { code: 'PS', name: 'パレスチナ' }, { code: 'YE', name: 'イエメン' }, { code: 'OM', name: 'オマーン' }, { code: 'BH', name: 'バーレーン' }, { code: 'MR', name: 'モーリタニア' },
  { code: 'SN', name: 'セネガル' }, { code: 'GM', name: 'ガンビア' }, { code: 'GN', name: 'ギニア' }, { code: 'GW', name: 'ギニアビサウ' }, { code: 'SL', name: 'シエラレオネ' },
  { code: 'LR', name: 'リベリア' }, { code: 'CI', name: 'コートジボワール' }, { code: 'GH', name: 'ガーナ' }, { code: 'TG', name: 'トーゴ' }, { code: 'BJ', name: 'ベナン' },
  { code: 'BF', name: 'ブルキナファソ' }, { code: 'NE', name: 'ニジェール' }, { code: 'TD', name: 'チャド' }, { code: 'CM', name: 'カメルーン' }, { code: 'CF', name: '中央アフリカ' },
  { code: 'GQ', name: '赤道ギニア' }, { code: 'GA', name: 'ガボン' }, { code: 'CG', name: 'コンゴ共和国' }, { code: 'CD', name: 'コンゴ民主共和国' }, { code: 'AO', name: 'アンゴラ' },
  { code: 'ZM', name: 'ザンビア' }, { code: 'MW', name: 'マラウイ' }, { code: 'MZ', name: 'モザンビーク' }, { code: 'ZW', name: 'ジンバブエ' }, { code: 'BW', name: 'ボツワナ' },
  { code: 'NA', name: 'ナミビア' }, { code: 'SZ', name: 'スワジランド' }, { code: 'LS', name: 'レソト' }, { code: 'MG', name: 'マダガスカル' }, { code: 'KM', name: 'コモロ' },
  { code: 'SC', name: 'セーシェル' }, { code: 'MU', name: 'モーリシャス' }, { code: 'CV', name: 'カーボベルデ' }, { code: 'SD', name: 'スーダン' }, { code: 'SS', name: '南スーダン' },
  { code: 'ET', name: 'エチオピア' }, { code: 'SO', name: 'ソマリア' }, { code: 'DJ', name: 'ジブチ' }, { code: 'ER', name: 'エリトリア' }, { code: 'TZ', name: 'タンザニア' },
  { code: 'UG', name: 'ウガンダ' }, { code: 'RW', name: 'ルワンダ' }, { code: 'BI', name: 'ブルンジ' }, { code: 'ST', name: 'サントメ・プリンシペ' }
]
const countryListEn = [
  { code: 'JP', name: 'Japan' }, { code: 'US', name: 'United States' }, { code: 'GB', name: 'United Kingdom' }, { code: 'CA', name: 'Canada' }, { code: 'AU', name: 'Australia' },
  { code: 'DE', name: 'Germany' }, { code: 'FR', name: 'France' }, { code: 'IT', name: 'Italy' }, { code: 'ES', name: 'Spain' }, { code: 'CN', name: 'China' },
  { code: 'KR', name: 'South Korea' }, { code: 'RU', name: 'Russia' }, { code: 'IN', name: 'India' }, { code: 'BR', name: 'Brazil' }, { code: 'MX', name: 'Mexico' },
  { code: 'ID', name: 'Indonesia' }, { code: 'TR', name: 'Turkey' }, { code: 'SA', name: 'Saudi Arabia' }, { code: 'ZA', name: 'South Africa' }, { code: 'EG', name: 'Egypt' },
  { code: 'AR', name: 'Argentina' }, { code: 'UA', name: 'Ukraine' }, { code: 'SE', name: 'Sweden' }, { code: 'NO', name: 'Norway' }, { code: 'DK', name: 'Denmark' },
  { code: 'FI', name: 'Finland' }, { code: 'PL', name: 'Poland' }, { code: 'NL', name: 'Netherlands' }, { code: 'BE', name: 'Belgium' }, { code: 'CH', name: 'Switzerland' },
  { code: 'AT', name: 'Austria' }, { code: 'GR', name: 'Greece' }, { code: 'CZ', name: 'Czech Republic' }, { code: 'HU', name: 'Hungary' }, { code: 'PT', name: 'Portugal' },
  { code: 'IE', name: 'Ireland' }, { code: 'NZ', name: 'New Zealand' }, { code: 'SG', name: 'Singapore' }, { code: 'TH', name: 'Thailand' }, { code: 'MY', name: 'Malaysia' },
  { code: 'PH', name: 'Philippines' }, { code: 'VN', name: 'Vietnam' }, { code: 'IL', name: 'Israel' }, { code: 'AE', name: 'United Arab Emirates' }, { code: 'QA', name: 'Qatar' },
  { code: 'KW', name: 'Kuwait' }, { code: 'CL', name: 'Chile' }, { code: 'CO', name: 'Colombia' }, { code: 'PE', name: 'Peru' }, { code: 'VE', name: 'Venezuela' },
  { code: 'PK', name: 'Pakistan' }, { code: 'BD', name: 'Bangladesh' }, { code: 'NG', name: 'Nigeria' }, { code: 'KE', name: 'Kenya' }, { code: 'MA', name: 'Morocco' },
  { code: 'TN', name: 'Tunisia' }, { code: 'DZ', name: 'Algeria' }, { code: 'RO', name: 'Romania' }, { code: 'BG', name: 'Bulgaria' }, { code: 'HR', name: 'Croatia' },
  { code: 'SK', name: 'Slovakia' }, { code: 'SI', name: 'Slovenia' }, { code: 'RS', name: 'Serbia' }, { code: 'BA', name: 'Bosnia and Herzegovina' }, { code: 'MK', name: 'North Macedonia' },
  { code: 'ME', name: 'Montenegro' }, { code: 'AL', name: 'Albania' }, { code: 'LT', name: 'Lithuania' }, { code: 'LV', name: 'Latvia' }, { code: 'EE', name: 'Estonia' },
  { code: 'BY', name: 'Belarus' }, { code: 'GE', name: 'Georgia' }, { code: 'AM', name: 'Armenia' }, { code: 'AZ', name: 'Azerbaijan' }, { code: 'KZ', name: 'Kazakhstan' },
  { code: 'UZ', name: 'Uzbekistan' }, { code: 'KG', name: 'Kyrgyzstan' }, { code: 'TJ', name: 'Tajikistan' }, { code: 'TM', name: 'Turkmenistan' }, { code: 'MD', name: 'Moldova' },
  { code: 'LU', name: 'Luxembourg' }, { code: 'LI', name: 'Liechtenstein' }, { code: 'MC', name: 'Monaco' }, { code: 'SM', name: 'San Marino' }, { code: 'VA', name: 'Vatican City' },
  { code: 'AD', name: 'Andorra' }, { code: 'MT', name: 'Malta' }, { code: 'CY', name: 'Cyprus' }, { code: 'IS', name: 'Iceland' }, { code: 'PY', name: 'Paraguay' },
  { code: 'UY', name: 'Uruguay' }, { code: 'BO', name: 'Bolivia' }, { code: 'EC', name: 'Ecuador' }, { code: 'GT', name: 'Guatemala' }, { code: 'HN', name: 'Honduras' },
  { code: 'SV', name: 'El Salvador' }, { code: 'NI', name: 'Nicaragua' }, { code: 'CR', name: 'Costa Rica' }, { code: 'PA', name: 'Panama' }, { code: 'CU', name: 'Cuba' },
  { code: 'DO', name: 'Dominican Republic' }, { code: 'HT', name: 'Haiti' }, { code: 'JM', name: 'Jamaica' }, { code: 'BS', name: 'Bahamas' }, { code: 'BB', name: 'Barbados' },
  { code: 'TT', name: 'Trinidad and Tobago' }, { code: 'GD', name: 'Grenada' }, { code: 'LC', name: 'Saint Lucia' }, { code: 'VC', name: 'Saint Vincent and the Grenadines' }, { code: 'AG', name: 'Antigua and Barbuda' },
  { code: 'KN', name: 'Saint Kitts and Nevis' }, { code: 'BZ', name: 'Belize' }, { code: 'GY', name: 'Guyana' }, { code: 'SR', name: 'Suriname' }, { code: 'FJ', name: 'Fiji' },
  { code: 'PG', name: 'Papua New Guinea' }, { code: 'SB', name: 'Solomon Islands' }, { code: 'VU', name: 'Vanuatu' }, { code: 'WS', name: 'Samoa' }, { code: 'TO', name: 'Tonga' },
  { code: 'TV', name: 'Tuvalu' }, { code: 'NR', name: 'Nauru' }, { code: 'PW', name: 'Palau' }, { code: 'FM', name: 'Micronesia' }, { code: 'MH', name: 'Marshall Islands' },
  { code: 'TL', name: 'East Timor' }, { code: 'KH', name: 'Cambodia' }, { code: 'LA', name: 'Laos' }, { code: 'MM', name: 'Myanmar' }, { code: 'NP', name: 'Nepal' },
  { code: 'LK', name: 'Sri Lanka' }, { code: 'MV', name: 'Maldives' }, { code: 'BT', name: 'Bhutan' }, { code: 'MN', name: 'Mongolia' }, { code: 'AF', name: 'Afghanistan' },
  { code: 'IR', name: 'Iran' }, { code: 'IQ', name: 'Iraq' }, { code: 'SY', name: 'Syria' }, { code: 'JO', name: 'Jordan' }, { code: 'LB', name: 'Lebanon' },
  { code: 'PS', name: 'Palestine' }, { code: 'YE', name: 'Yemen' }, { code: 'OM', name: 'Oman' }, { code: 'BH', name: 'Bahrain' }, { code: 'MR', name: 'Mauritania' },
  { code: 'SN', name: 'Senegal' }, { code: 'GM', name: 'Gambia' }, { code: 'GN', name: 'Guinea' }, { code: 'GW', name: 'Guinea-Bissau' }, { code: 'SL', name: 'Sierra Leone' },
  { code: 'LR', name: 'Liberia' }, { code: 'CI', name: 'Ivory Coast' }, { code: 'GH', name: 'Ghana' }, { code: 'TG', name: 'Togo' }, { code: 'BJ', name: 'Benin' },
  { code: 'BF', name: 'Burkina Faso' }, { code: 'NE', name: 'Niger' }, { code: 'TD', name: 'Chad' }, { code: 'CM', name: 'Cameroon' }, { code: 'CF', name: 'Central African Republic' },
  { code: 'GQ', name: 'Equatorial Guinea' }, { code: 'GA', name: 'Gabon' }, { code: 'CG', name: 'Republic of the Congo' }, { code: 'CD', name: 'Democratic Republic of the Congo' }, { code: 'AO', name: 'Angola' },
  { code: 'ZM', name: 'Zambia' }, { code: 'MW', name: 'Malawi' }, { code: 'MZ', name: 'Mozambique' }, { code: 'ZW', name: 'Zimbabwe' }, { code: 'BW', name: 'Botswana' },
  { code: 'NA', name: 'Namibia' }, { code: 'SZ', name: 'Eswatini' }, { code: 'LS', name: 'Lesotho' }, { code: 'MG', name: 'Madagascar' }, { code: 'KM', name: 'Comoros' },
  { code: 'SC', name: 'Seychelles' }, { code: 'MU', name: 'Mauritius' }, { code: 'CV', name: 'Cape Verde' }, { code: 'SD', name: 'Sudan' }, { code: 'SS', name: 'South Sudan' },
  { code: 'ET', name: 'Ethiopia' }, { code: 'SO', name: 'Somalia' }, { code: 'DJ', name: 'Djibouti' }, { code: 'ER', name: 'Eritrea' }, { code: 'TZ', name: 'Tanzania' },
  { code: 'UG', name: 'Uganda' }, { code: 'RW', name: 'Rwanda' }, { code: 'BI', name: 'Burundi' }, { code: 'ST', name: 'São Tomé and Príncipe' }
]
const countryList = computed(() => lang.value === 'ja' ? countryListJa : countryListEn)

const form = ref({
  name: '',
  headline: '',
  subHeadline: '',
  hashtags: '',
  references: '',
  aiReplyPattern: '',
  aiSummary: 'on',
  likes: 'show',
  saves: 'show',
  comments: 'on',
  payment: 'on',
  publishTo: 'domestic',
  selectedCountries: ["", "", ""] as string[],
})

const errors = ref({
  name: false,
  headline: false,
  subHeadline: false,
  hashtags: false,
})
const errorMsg = ref('')

const showPublishTime = ref(false)
const publishTime = ref<string>('')

const fakeTime = computed(() => {
  // 架空の時間を生成（例：07月15日12時34分 or 07/15 12:34）
  if (lang.value === 'ja') {
    return '07月15日12時34分'
  } else {
    return '07/15 12:34'
  }
})

function onPublishToChange() {
  if (form.value.publishTo !== 'specific') {
    form.value.selectedCountries = ["", "", ""]
  }
}

function onClose() {
  alert(lang.value === 'ja' ? 'ウィンドウを閉じます' : 'Close window')
}

function onExport() {
  alert(lang.value === 'ja' ? '書き出し機能は未実装です' : 'Export feature is not implemented yet')
}

const canUpload = computed(() => {
  // 必須項目チェック
  if (!form.value.name || !form.value.headline || !form.value.subHeadline) return false
  // ハッシュタグはカンマ区切りで1つ以上必須
  const hashtagsArr = form.value.hashtags.split(',').map(s => s.trim()).filter(Boolean)
  if (hashtagsArr.length === 0) return false
  if (form.value.publishTo === 'specific') {
    const selected = form.value.selectedCountries.filter(c => c)
    const unique = new Set(selected)
    if (selected.length === 0) return false
    if (selected.length !== unique.size) return false
  }
  return true
})

function onUpload() {
  // バリデーション
  errors.value = { name: false, headline: false, subHeadline: false, hashtags: false }
  errorMsg.value = ''
  if (!form.value.name) errors.value.name = true
  if (!form.value.headline) errors.value.headline = true
  if (!form.value.subHeadline) errors.value.subHeadline = true
  // ハッシュタグはカンマ区切りで1つ以上必須
  const hashtagsArr = form.value.hashtags.split(',').map(s => s.trim()).filter(Boolean)
  if (hashtagsArr.length === 0) errors.value.hashtags = true
  if (errors.value.name || errors.value.headline || errors.value.subHeadline || errors.value.hashtags) {
    errorMsg.value = t('errorRequired')
    if (errors.value.hashtags) errorMsg.value = t('errorHashtag')
    return
  }
  if (form.value.publishTo === 'specific') {
    const selected = form.value.selectedCountries.filter(c => c)
    const unique = new Set(selected)
    if (selected.length === 0) {
      errorMsg.value = t('errorCountry')
      return
    }
    if (selected.length !== unique.size) {
      errorMsg.value = t('errorCountryDup')
      return
    }
  }
  errorMsg.value = ''
  const now = new Date()
  const mm = String(now.getMonth() + 1).padStart(2, '0')
  const dd = String(now.getDate()).padStart(2, '0')
  const hh = String(now.getHours()).padStart(2, '0')
  const min = String(now.getMinutes()).padStart(2, '0')
  publishTime.value = lang.value === 'ja'
    ? `${mm}月${dd}日${hh}時${min}分`
    : `${mm}/${dd} ${hh}:${min}`
  showPublishTime.value = true
}
</script>

<style scoped>
.post-settings-root {
  width: 100vw;
  height: 100vh;
  background: #ededed;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
}
.panel-width {
  width: 1450px;
  max-width: 99vw;
}
.bar-balance {
  margin-left: auto;
  margin-right: auto;
}
.lang-switcher {
  width: 900px;
  max-width: 96vw;
  display: flex;
  justify-content: flex-end;
  margin-top: 16px;
  margin-bottom: 0;
}
.lang-switcher button {
  background: #fff;
  border: 1px solid #bbb;
  color: #222;
  padding: 4px 16px;
  margin-left: 8px;
  border-radius: 4px 4px 0 0;
  font-size: 1rem;
  cursor: pointer;
  outline: none;
  transition: background 0.2s;
}
.lang-switcher button.active {
  background: #1976d2;
  color: #fff;
  border-bottom: 2px solid #1976d2;
}
.window-bar {
  height: 36px;
  background: #222;
  border-top-left-radius: 8px;
  border-top-right-radius: 8px;
  display: flex;
  align-items: center;
  position: relative;
  margin-top: 0;
}
.window-bar-btns {
  display: flex;
  gap: 6px;
  margin-left: 16px;
}
.bar-btn {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  display: inline-block;
}
.bar-btn.red { background: #ff5f56; border: 1px solid #e0443e; }
.bar-btn.yellow { background: #ffbd2e; border: 1px solid #dea123; }
.bar-btn.green { background: #27c93f; border: 1px solid #13a10e; }
.window-bar-title {
  position: absolute;
  left: 0; right: 0;
  text-align: center;
  color: #fff;
  font-size: 1.1rem;
  font-weight: 500;
  letter-spacing: 2px;
}
.post-settings-panel {
  background: #ffffff;
  border-bottom-left-radius: 8px;
  border-bottom-right-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
  display: flex;
  min-width: 1450px;
  max-width: 99vw;
  min-height: 850px;
  padding: 48px 36px 36px 36px;
  box-sizing: border-box;
  justify-content: flex-start; /* Align to start to keep left panel fixed */
  align-items: center; /* Center vertically */
}
.post-settings-left {
  flex: none; /* Don't flex, keep fixed width */
  width: 452px; /* Fixed width: 380px (image) + 36px (left padding) + 36px (right padding) */
  display: flex;
  align-items: center;
  justify-content: center;
}
.image-upload-placeholder {
  width: 380px; /* Increased width */
  height: 537px; /* A4 ratio: width:height = 1:1.414 (380 * 1.414 ≈ 537) */
  background: #fff;
  border: 2px solid #e0e0e0;
  box-sizing: border-box;
}
.post-settings-right {
  flex: 1;
  padding-left: 36px;
  display: flex;
  flex-direction: column;
  min-width: 0;
  margin-top: 0; /* Remove top margin since parent is now centered */
  margin-bottom: 0; /* Remove bottom margin since parent is now centered */
}
form {
  width: 100%;
  min-width: 0;
}
.form-group {
  margin-bottom: 55px; /* Further increased margin for better spacing */
  display: flex;
  align-items: center;
  min-width: 0;
}
.form-group label {
  width: 220px; /* Even wider to prevent any text wrapping */
  font-weight: bold;
  color: #222;
  font-size: 1rem; /* Increased font size */
  flex-shrink: 0;
  min-width: 0;
  white-space: nowrap; /* Prevent line breaks */
}
.required-asterisk {
  color: #ff3c3c;
  margin-left: 2px;
  font-size: 1.1em;
  font-weight: bold;
}
.form-group input[type="text"] {
  flex: 1;
  border: 1px solid #bdbdbd;
  border-radius: 0;
  padding: 7px 10px;
  font-size: 1rem;
  background: #fafafa;
  color: #222;
  margin-left: 8px; /* Restored margin */
  min-width: 0;
  max-width: 100%;
  box-sizing: border-box;
  width: 100%;
}
.input-error {
  border: 2px solid #ff3c3c !important;
  background: #fff0f0 !important;
}
.form-group input[type="text"]::placeholder {
  color: #bdbdbd;
  white-space: nowrap; /* Prevent line breaks */
  text-overflow: ellipsis;
  overflow: hidden;
}
.form-row.select-row {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 55px; /* Increased margin for better spacing */
}
.form-group.small {
  flex: none;
  width: auto;
  min-width: 100px;
  margin-bottom: 0;
}
.form-group.small label {
  width: auto;
  font-size: 1rem; /* Increased font size */
  margin-right: 4px;
}
.form-group.small select {
  width: 80px;
  font-size: 1rem; /* Increased font size */
  padding: 4px 8px; /* Increased padding */
  border-radius: 0;
  border: 1px solid #bdbdbd;
  background: #fafafa;
  color: #222;
}
.publish-select {
  width: 140px; /* Increased width to show full text */
  font-size: 1rem;
  padding: 5px 8px; /* Increased padding */
  border-radius: 0;
  border: 1px solid #bdbdbd;
  background: #fafafa;
  color: #222;
  margin-left: 0;
}
.publish-section {
  display: flex;
  margin-bottom: 55px; /* Match other form groups */
}

.publish-main {
  display: flex;
  align-items: center;
  margin-bottom: 20px; /* Add space between main dropdown and country selects */
}

.publish-main label {
  width: 68px; /* Match other labels */
  font-weight: bold;
  color: #222;
  font-size: 1rem;
  flex-shrink: 0;
  min-width: 0;
  white-space: nowrap;
}

.publish-countries {
  display: flex;
  justify-content: flex-start;
  gap: 40px; /* Equal spacing between country selects */
  margin-left: 115px; /* Align with the main dropdown */
  align-items: flex-start;
}
.country-select-item {
  display: flex;
  align-items: center;
  gap: 3px; /* Consistent gap between label and select */
  flex: none; /* Don't flex, use fixed width */
  justify-content: flex-start;
  width: 155px; /* Fixed width for each country select item */
}
.country-select-item label {
  font-weight: normal;
  font-size: 1rem;
  width: 65px;
  text-align: left;
}
.country-select-item select {
  width: 80px; /* Match the width of other small selects */
  font-size: 1rem;
  padding: 4px 8px;
  border-radius: 0;
  border: 1px solid #bdbdbd;
  background: #fafafa;
  color: #222;
}
.error {
  color: #d00;
  font-size: 0.9em;
  margin-left: 8px;
}
.error-msg {
  color: #d00;
  font-size: 1.05em;
  margin: 8px 0 0 168px;
  font-weight: bold;
}
.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 18px;
  margin-top: 15px; /* Reduced margin to eliminate orange box space */
}
.close-btn {
  background: #ededed;
  color: #222;
  border: 1px solid #bdbdbd;
  border-radius: 4px;
  padding: 8px 28px;
  font-size: 1rem;
}
.export-btn {
  background: #1976d2;
  color: #fff;
  border: none;
  border-radius: 4px;
  padding: 8px 28px;
  font-size: 1rem;
}
.upload-btn {
  background: #222;
  color: #fff;
  border: 2px solid #ff3c3c;
  border-radius: 4px;
  padding: 8px 32px;
  font-size: 1rem;
  font-weight: bold;
  box-shadow: none;
  transition: background 0.2s;
}
.upload-btn:hover {
  background: #ff3c3c;
  color: #fff;
}
.upload-btn[disabled] {
  opacity: 0.5;
  cursor: not-allowed;
}
.publish-time-msg {
  margin-top: 24px;
  color: #222;
  font-size: 1.1rem;
  text-align: center;
  font-weight: bold;
}
</style> 