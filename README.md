# i18n-enhancer

> Make i18n 🌍 joyful 🤗 as it deserved to be!

An npm package **enhances** `react-i18next` and `i18next`, enhances the functionality of `useTranslate` by providing precise types, thus **hinting keys and interpolation variable names** within translated text, among other enhancements.

## Installation

```bash
npm install react-i18next i18next --save

npm install react-i18next-enhancer --save
```

## Usage

### For `react-i18next`

#### 1. initialize `react-i18next`

```typescript
// src/locales/i18n.ts

import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import { getLast } from 'i18n-enhancer/utils';

import zhLocale from './zh/index.ts';
import enLocale from './en/index.ts';

i18n.use(initReactI18next).init({
  resources: {
    en: {
      translation: enLocale,
    },
    zh: {
      translation: zhLocale,
    },
  },
  lng: 'zh',
  fallbackLng: 'zh',

  interpolation: {
    escapeValue: false,
  },

  // 当翻译键不存在时，返回键最后一部分作为默认值，意味着如果你用中文做键那么你无需再添加中文翻译
  // When the translation key is missing, return the last part of the key as the default value,
  // which means if you use Chinese as the key, you don't need to add a Chinese translation.
  parseMissingKeyHandler: (key, defaultValue) => {
    return defaultValue ?? getLast(key);
  },
});

export default i18n;
```

#### 2. Create a `react-i18next` **enhancer**

```typescript
import React from 'react';
import * as reactI18N from 'react-i18next';
import { enhance } from 'i18n-enhancer/react-i18next';

import initializedI18N from '@/locales/i18n';

// 只需要 import 英文即可，因为我们一般使用中文开发，如此反而可以检测出没有翻译的 key
// 其次中文一般无需翻译，因为可以直接 key 中的中文
// 故此处只import英文
// Just import English, and it can detect untranslated keys;
// Chinese generally doesn't need to be translated since it can be directly used from the keys in Chinese. Thus, only import English here.
type ITranslationsEn = typeof import('./en').default;

const enhancer = enhance({
  reactI18N,
  i18n: initializedI18N,
});
```

### 3. Use `useT` instead of `useTranslation` in your components

> `useT` is type enhanced version of `useTranslation` with the type of your translated text as the key type.

```typescript
const Shopping: React.FC = () => {
  const { useT } = enhancer;
  const { t, i18n } = useT<ITranslationsEn>();

  // i18n.changeLanguage('en');

  return (<main>
    <div>{t('shopping.去支付')}</div>
    <div>{t('shopping.orderSummaryText')}</div>
    <div>{t('shopping.总共', { total: 100 })}</div>
  </main>);
};
```

### 4. Magic happens

假如我们的翻译文本如下：
Suppose our translation text is as follows:

```typescript
// en
{
  'shopping.去支付': 'Checkout',
  'shopping.orderSummaryText': 'Please review your order details below.',
  'shopping.总共': 'Total {{ total }} RMB',
} as const

// zh
{
  // 'shopping.去支付': '去支付', // we use `getLast` to extract text from key so no need to provide translation for Chinese. What a efficient way!
  'shopping.orderSummaryText': '请确认您的订单信息',
  'shopping.总共': '总共 {{ total }} 元',
} as const
```

当你输入 `t('shopping.` 你会看到所有翻译的 key 都会提示出来。

并且如果你的光标悬浮到 `t` 函数上，你会看到如下提示：

If you hover over the `t` function, you will see the following prompts:

```typescript
t('shopping.去支付'): 'Checkout'; 
t('shopping.orderSummaryText'): 'Please review your order details below.';
```

**Yes it reflects the translation** instead of the just `string`! (Of course, `as const` must applied).

How about the interpolation variables?

```typescript
t('shopping.总共', { total: number }): 'Total {{ total }} RMB';
```

Yes again the interpolation variables are hinted as well.

**It is truly i18n type safe.**

Certainly `<Shopping />` will rendered as expected:

for English:

```jsx
<main>
  <div>Checkout</div>
  <div>Please review your order details below.</div>
  <div>Total 100 RMB</div>
</main>
```

for Chinese:

```jsx
<main>
  <div>去支付</div>
  <div>请确认您的订单信息</div>
  <div>总共 100 元</div>
</main>
```
