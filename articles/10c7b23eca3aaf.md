---
title: 'みなさん画像のinputコンポーネントどう作ってます？'
emoji: '💨'
type: 'tech'
topics: ['Nuxt3', 'vue', 'vuetify']
published: true
---

### ぼやき

年末なので長文書く時間がなくて小ネタでお茶を濁したい....

### 本題

みなさん、画像のinputコンポーネントどのように作っていらっしゃいますか？

ただファイル選ぶだけなら単なる`<input type="file" />`でも良いんですが、もう少し使いやすいコンポーネントにしたいと思うと結構考えることが増えます。

- プレビュー機能
- ドラッグ＆ドロップ対応
- ファイル名表示
- ホバー時対応

これらをまとめて解決するコンポーネントを作ったので共有したいと思います。

### 作ったもの

デザインはSmartHR Design Systemより[Drop Zone🔗](https://smarthr.design/products/components/drop-zone/)の実装を参考にさせて頂きました。

また、実際に個人開発で使うときはS3に上げるファイルを扱っています。
下記は、その実コードをS3関係なく使えるように多少一般化したものになります。
vuetifyのcss classやcomponentを使っていますが皆様の環境に応じて適宜読み替えてください

```vue:atom-input-file.vue
<script setup lang="ts">
const props = withDefaults(defineProps<{ modelValue: File | string | null }>(), {
  modelValue: null
})
const emit = defineEmits<{
  (e: 'update:model-value', value: any): void
}>()
const resetFileObject = () => {
  files.value = []
  imageURL.value = ''
  emit('update:model-value', null)
}
const emitImg = async () => {
  if (!files.value.length) return
  emit('update:model-value', files.value[0])
}
const onImagePicked = async (e: Event) => {
  const images = (e.target as HTMLInputElement)?.files
  if (!images) return
  if (Array.from(images).some((v) => v.size >= 5 * 1024 * 1024)) {
    alert('アップロード可能な画像サイズは5MBまでです')
    return
  }
  if (images) files.value = [...images]
  await emitImg()
}
const typeSafetyImage = async (
  file: string | File | null
): Promise<string> => {
  if (!file) return ''
  else if (typeof file === 'string') return file
  else if (file instanceof File) return URL.createObjectURL(file)
}
const onDrop = async (e: DragEvent) => {
  if (!e.dataTransfer?.files) return
  files.value = [...e.dataTransfer.files]
  await emitImg()
  isDropOvering.value = false
}
const onDragOver = (e: DragEvent) => {
  isDropOvering.value = true
}
const onDragLeave = (e: DragEvent) => {
  isDropOvering.value = false
}
const isDropOvering = ref<boolean>(false)
const files = ref<File[]>([])
const imageURL = ref<string>('')
imageURL.value = await typeSafetyImage(props.modelValue)
watch(props, async (_, c) => {
  imageURL.value = await typeSafetyImage(props.modelValue)
})
</script>
<template>
  <div
    class="w-100 height-200 bg-grey-lighten-4 rounded border-width-1 border-dotted border-grey-lighten-1 position-relative"
  >
    <v-img v-if="imageURL" :src="imageURL" :aspect-ratio="16 / 9" class="max-height-198" />
    <v-hover v-slot="{ isHovering, props: hover }">
      <div
        class="w-100 h-100 pa-10 d-flex flex-column justify-center align-center position-absolute"
        :class="[
          { 'bg-grey-darken-2': isDropOvering || (imageURL && isHovering) },
          isDropOvering || (imageURL && isHovering)
            ? 'opacity-dot8'
            : imageURL
              ? 'opacity-dot0'
              : 'opacity-dot10'
        ]"
        v-bind="hover"
        @drop.stop.prevent="onDrop"
        @dragover.stop.prevent="onDragOver"
        @dragleave.stop.prevent="onDragLeave"
      >
        <atom-text
          text="ここにドラッグ&ドロップ"
          :color="imageURL || isDropOvering ? 'text-white' : 'text-grey-darken-1'"
          line-height="line-height-lg"
          class="mb-4"
        />
        <atom-text
          text="または"
          :color="imageURL || isDropOvering ? 'text-white' : 'text-grey-darken-1'"
          line-height="line-height-lg"
          class="mb-4"
        />
        <atom-button
          v-if="imageURL"
          :text="`リセット（ ${modelValue?.name || ''} ）`"
          icon="mdi-close"
          class="bg-white rounded border-solid border-width-1 border-grey-lighten-1"
          @click="resetFileObject"
        />
        <label
          v-else
          class="px-4 py-2 d-flex align-center bg-white rounded border-solid border-width-1 border-grey-lighten-1 cursor-pointer"
        >
          <v-icon icon="mdi-folder-open" class="mr-2" />
          <atom-text text="ファイルを選択" />
          <input
            type="file"
            accept="image/png, image/jpeg, image/gif"
            class="d-none"
            @input="onImagePicked($event)"
          />
        </label>
      </div>
    </v-hover>
  </div>
</template>
```

```vue:atom-text.vue
<script setup lang="ts">
withDefaults(
  defineProps<{
    comp?: string
    text?: string | null
    fontSize?: string
    fontWeight?: string
    color?: string
    lineHeight?: string
  }>(),
  {
    comp: 'p',
    fontSize: 'text-subtitle-1',
    fontWeight: 'font-weight-bold',
    color: 'text-black',
    lineHeight: 'line-height-sm'
  }
)
</script>
<template>
  <component :is="comp" :class="[fontSize, fontWeight, color, lineHeight]">
    {{ text }}
    <slot />
  </component>
</template>
```

```vue:atom-button.vue
<script setup lang="ts">
withDefaults(
  defineProps<{
    text: string
    icon: string
    src: string
    disabled: boolean
    variant: 'small' | 'medium' | 'large'
  }>(),
  {
    text: '',
    icon: '',
    src: '',
    disabled: false,
    variant: 'medium'
  }
)
const isHovering = ref<boolean>(false)
</script>
<template>
  <button
    type="button"
    :disabled="disabled"
    class="px-4 py-2 d-flex align-center"
    :class="[disabled ? 'opacity-dot3 cursor-not-allowed' : { 'opacity-dot7': isHovering }]"
    @mouseenter="isHovering = true"
    @mouseleave="isHovering = false"
  >
    <slot name="prepend">
      <v-icon
        v-if="icon"
        :icon="icon"
        :size="variant === 'large' ? '24w' : variant === 'medium' ? '21' : '18'"
        :class="[{ 'mr-1': text }]"
      />
      <v-img
        v-if="src"
        :src="src"
        class="mr-2 flex-0"
        :class="
          variant === 'large'
            ? 'width-16 height-16'
            : variant === 'medium'
            ? 'width-14 height-14'
            : 'width-12  height-12'
        "
      />
    </slot>
    <slot name="center">
      <span
        :class="[
          variant === 'large'
            ? 'text-subtitle-1'
            : variant === 'medium'
            ? 'text-subtitle-2'
            : 'text-caption',
          'line-height-lg font-weight-bold'
        ]"
      >
        {{ text }}
      </span>
    </slot>
    <slot name="append" />
    <slot />
  </button>
</template>
```

### 少し説明

デフォルトでは説明とファイル選択ボタンのみの表示です。

ファイル選択時には5MB制限のバリデーションを掛け、問題なければemitで親コンポーネントに情報を渡します。

ファイル選択後は、ホバーで背景が黒くなりリセットボタンが出現するようにしています。

また、基本的なドラッグ&ドロップ機能を実装の上、`isDropOvering`フラグでファイルをドロップしようとドラッグオーバーした際に、ホバー時と同様に背景を黒くしてドラッグオーバーに反応していることを視覚的に分かりやすくしています。

### おわりに

このコンポーネントは考えることが尽きなくて無限に時間が溶けていきます。。。。

複数ファイル対応やpdf対応などまだまだ対応していくべきことは残っているので改善し続けていきたいと思います。
