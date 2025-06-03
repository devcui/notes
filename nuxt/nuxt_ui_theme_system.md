# Nuxt UI Theme System: Button TV (Tailwind Variants) 详尽分析

## 概述

本文将以极其详细的方式分析 Nuxt UI 中 Button 组件的 TV (Tailwind Variants) 系统，包含每一步的输入输出、数据转换过程和实际运行示例。我们将追踪一个完整的请求流程：从用户写下 `<UButton color="primary" variant="solid" size="md">Click me</UButton>` 到最终生成 CSS 类名的全过程。

## 1. 模块初始化阶段：数据输入与处理

### 1.1 用户配置输入

**输入：用户在 `nuxt.config.ts` 中的配置**
```typescript
export default defineNuxtConfig({
  modules: ['@nuxt/ui'],
  ui: {
    theme: {
      colors: ['primary', 'brand', 'accent'],
      transitions: true
    }
  }
})
```

**处理过程：模块初始化 (`module.ts` 第 84 行)**
```typescript
async setup(options, nuxt) {
  const { resolve } = createResolver(import.meta.url)
  
  // 第一步：颜色解析
  // 输入: options.theme.colors = ['primary', 'brand', 'accent']
  options.theme.colors = resolveColors(options.theme.colors)
  // 输出: ['primary', 'brand', 'accent'] (因为包含了 primary)
  
  // 第二步：设置 nuxt 配置
  nuxt.options.ui = options
  // 输出: { prefix: 'U', fonts: true, colorMode: true, theme: { colors: [...], transitions: true } }
}
```

**`resolveColors` 函数详细处理 (`defaults.ts` 第 25 行)**
```typescript
export const resolveColors = (colors?: string[]) => {
  return colors?.length
    ? [...new Set(['primary', ...colors])]  // 确保 primary 始终存在
    : ['primary', 'secondary', 'success', 'info', 'warning', 'error']
}

// 示例执行：
// 输入: ['primary', 'brand', 'accent']
// 处理: ['primary', 'primary', 'brand', 'accent'] -> 去重
// 输出: ['primary', 'brand', 'accent']
```

### 1.2 AppConfig 注入过程

**输入处理：`getDefaultUiConfig` 函数 (`defaults.ts` 第 6 行)**
```typescript
export const getDefaultUiConfig = (colors?: string[]) => ({
  colors: pick({
    primary: 'green',      // 默认颜色映射
    secondary: 'blue',
    success: 'green',
    info: 'blue',
    warning: 'yellow',
    error: 'red',
    neutral: 'slate'
  }, [...(colors || []), 'neutral' as any]),
  icons
})

// 执行过程：
// 输入: colors = ['primary', 'brand', 'accent']
// pick 函数选择: ['primary', 'brand', 'accent', 'neutral']
// 输出: {
//   colors: {
//     primary: 'green',
//     neutral: 'slate'
//     // brand 和 accent 不在默认映射中，所以不包含
//   },
//   icons: { /* 图标配置 */ }
// }
```

**AppConfig 合并过程 (`module.ts` 第 90 行)**
```typescript
nuxt.options.appConfig.ui = defu(
  nuxt.options.appConfig.ui || {}, 
  getDefaultUiConfig(options.theme.colors)
)

// 实际执行：
// 输入1: nuxt.options.appConfig.ui = {}
// 输入2: getDefaultUiConfig(['primary', 'brand', 'accent'])
// defu 合并结果:
// {
//   colors: {
//     primary: 'green',
//     neutral: 'slate'
//   },
//   icons: { plus: 'i-lucide-plus', minus: 'i-lucide-minus', ... }
// }
```

## 2. 主题定义阶段：Button 主题生成详解

### 2.1 Button 主题函数调用过程

**主题函数定义 (`theme/button.ts`)**
```typescript
import type { ModuleOptions } from '../module'
import { buttonGroupVariant } from './button-group'

export default (options: Required<ModuleOptions>) => {
  // 这里 options 是完整的模块配置
  console.log('Button theme 函数接收的 options:', options)
  // {
  //   prefix: 'U',
  //   fonts: true,
  //   colorMode: true,
  //   theme: {
  //     colors: ['primary', 'brand', 'accent'],
  //     transitions: true
  //   }
  // }
}
```

**动态颜色变体生成详解**
```typescript
// 第一步：颜色变体对象生成
const colorVariants = Object.fromEntries(
  (options.theme.colors || []).map((color: string) => [color, ''])
)

// 执行过程：
// 输入: options.theme.colors = ['primary', 'brand', 'accent']
// map 操作: [['primary', ''], ['brand', ''], ['accent', '']]
// Object.fromEntries 转换: { primary: '', brand: '', accent: '' }
// 添加 neutral: { primary: '', brand: '', accent: '', neutral: '' }

console.log('生成的颜色变体:', colorVariants)
// 输出: { primary: '', brand: '', accent: '', neutral: '' }
```

**复合变体生成的详细过程**
```typescript
// solid 变体的复合生成
const solidCompoundVariants = (options.theme.colors || []).map((color: string) => ({
  color,
  variant: 'solid',
  class: `text-inverted bg-${color} hover:bg-${color}/75 disabled:bg-${color} aria-disabled:bg-${color} focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-${color}`
}))

// 执行过程：
// 输入: ['primary', 'brand', 'accent']
// 输出: [
//   {
//     color: 'primary',
//     variant: 'solid', 
//     class: 'text-inverted bg-primary hover:bg-primary/75 disabled:bg-primary aria-disabled:bg-primary focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-primary'
//   },
//   {
//     color: 'brand',
//     variant: 'solid',
//     class: 'text-inverted bg-brand hover:bg-brand/75 disabled:bg-brand aria-disabled:bg-brand focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-brand'
//   },
//   {
//     color: 'accent', 
//     variant: 'solid',
//     class: 'text-inverted bg-accent hover:bg-accent/75 disabled:bg-accent aria-disabled:bg-accent focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-accent'
//   }
// ]
```

### 2.2 完整的主题对象输出

**最终生成的主题配置**
```typescript
// Button 主题函数的完整返回值
const buttonTheme = {
  slots: {
    base: [
      'rounded-md font-medium inline-flex items-center disabled:cursor-not-allowed aria-disabled:cursor-not-allowed disabled:opacity-75 aria-disabled:opacity-75',
      options.theme.transitions && 'transition-colors'  // 根据配置决定是否包含过渡
    ].filter(Boolean),  // 过滤掉 false 值
    label: 'truncate',
    leadingIcon: 'shrink-0',
    leadingAvatar: 'shrink-0',
    leadingAvatarSize: '',
    trailingIcon: 'shrink-0'
  },
  variants: {
    // buttonGroup 变体从 button-group.ts 导入
    buttonGroup: {
      horizontal: 'not-only:first:rounded-e-none not-only:last:rounded-s-none not-last:not-first:rounded-none focus-visible:z-[1]',
      vertical: 'not-only:first:rounded-b-none not-only:last:rounded-t-none not-last:not-first:rounded-none focus-visible:z-[1]'
    },
    // 动态生成的颜色变体
    color: {
      primary: '',
      brand: '',
      accent: '',
      neutral: ''
    },
    variant: {
      solid: '',
      outline: '',
      soft: '',
      subtle: '',
      ghost: '',
      link: ''
    },
    size: {
      xs: {
        base: 'px-2 py-1 text-xs gap-1',
        leadingIcon: 'size-4',
        leadingAvatarSize: '3xs',
        trailingIcon: 'size-4'
      },
      sm: {
        base: 'px-2.5 py-1.5 text-xs gap-1.5',
        leadingIcon: 'size-4',
        leadingAvatarSize: '3xs', 
        trailingIcon: 'size-4'
      },
      md: {
        base: 'px-2.5 py-1.5 text-sm gap-1.5',
        leadingIcon: 'size-5',
        leadingAvatarSize: '2xs',
        trailingIcon: 'size-5'
      },
      lg: {
        base: 'px-3 py-2 text-sm gap-2',
        leadingIcon: 'size-5',
        leadingAvatarSize: '2xs',
        trailingIcon: 'size-5'
      },
      xl: {
        base: 'px-3 py-2 text-base gap-2',
        leadingIcon: 'size-6',
        leadingAvatarSize: 'xs',
        trailingIcon: 'size-6'
      }
    },
    block: {
      true: {
        base: 'w-full justify-center',
        trailingIcon: 'ms-auto'
      }
    },
    square: { true: '' },
    leading: { true: '' },
    trailing: { true: '' },
    loading: { true: '' },
    active: {
      true: { base: '' },
      false: { base: '' }
    }
  },
  compoundVariants: [
    // 每种颜色 + solid 变体的组合 (3个)
    { color: 'primary', variant: 'solid', class: 'text-inverted bg-primary hover:bg-primary/75 disabled:bg-primary aria-disabled:bg-primary focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-primary' },
    { color: 'brand', variant: 'solid', class: 'text-inverted bg-brand hover:bg-brand/75 disabled:bg-brand aria-disabled:bg-brand focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-brand' },
    { color: 'accent', variant: 'solid', class: 'text-inverted bg-accent hover:bg-accent/75 disabled:bg-accent aria-disabled:bg-accent focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-accent' },
    
    // 每种颜色 + outline 变体的组合 (3个)
    { color: 'primary', variant: 'outline', class: 'ring ring-inset ring-primary/50 text-primary hover:bg-primary/10' },
    { color: 'brand', variant: 'outline', class: 'ring ring-inset ring-brand/50 text-brand hover:bg-brand/10' },
    { color: 'accent', variant: 'outline', class: 'ring ring-inset ring-accent/50 text-accent hover:bg-accent/10' },
    
    // 类似地，还有 soft, subtle, ghost, link 变体，每种 18个 复合变体
    // ...总共约 18 个动态生成的复合变体
    
    // neutral 颜色的特殊处理 (6个)
    { color: 'neutral', variant: 'solid', class: 'text-inverted bg-inverted hover:bg-inverted/90' },
    { color: 'neutral', variant: 'outline', class: 'ring ring-inset ring-accented text-default bg-default' },
    // ...其他 neutral 变体
    
    // square + size 的组合 (5个)
    { size: 'xs', square: true, class: 'p-1' },
    { size: 'sm', square: true, class: 'p-1.5' },
    { size: 'md', square: true, class: 'p-1.5' },
    { size: 'lg', square: true, class: 'p-2' },
    { size: 'xl', square: true, class: 'p-2' },
    
    // loading 相关的组合 (2个)
    { loading: true, leading: true, class: { leadingIcon: 'animate-spin' } },
    { loading: true, leading: false, trailing: true, class: { trailingIcon: 'animate-spin' } }
  ],
  defaultVariants: {
    color: 'primary',
    variant: 'solid', 
    size: 'md'
  }
}

console.log('Button 主题对象生成完成:', buttonTheme)
```

## 3. 模板生成阶段：构建时代码生成详解

### 3.1 模板生成函数执行过程

**`getTemplates` 函数调用 (`templates.ts` 第 19 行)**
```typescript
export function getTemplates(options: ModuleOptions, uiConfig: Record<string, any>) {
  // 输入参数：
  console.log('getTemplates 输入 options:', options)
  // {
  //   prefix: 'U',
  //   fonts: true, 
  //   colorMode: true,
  //   theme: { colors: ['primary', 'brand', 'accent'], transitions: true }
  // }
  
  console.log('getTemplates 输入 uiConfig:', uiConfig)
  // {
  //   colors: { primary: 'green', neutral: 'slate' },
  //   icons: { plus: 'i-lucide-plus', ... }
  // }

  const templates: NuxtTemplate[] = []
  
  // 遍历所有主题组件
  for (const component in theme) {
    // component = 'button', 'input', 'card', ...
    console.log('正在处理组件:', component)
  }
}
```

### 3.2 Button 模板生成的详细过程

**单个组件模板生成逻辑**
```typescript
// 针对 button 组件的处理
templates.push({
  filename: `ui/${kebabCase('button')}.ts`,  // 生成 'ui/button.ts'
  write: true,
  getContents: async () => {
    // 第一步：获取主题模板函数
    const template = (theme as any)['button']  // 这是我们之前定义的函数
    console.log('Button template 函数:', template)
    
    // 第二步：执行主题函数，传入完整的 options
    const result = typeof template === 'function' ? template(options) : template
    console.log('执行 button 主题函数后的结果:', result)
    // result 就是之前第二章分析的完整 buttonTheme 对象
    
    // 第三步：提取需要类型声明的变体
    const variants = Object.entries(result.variants || {})
      .filter(([_, values]) => {
        const keys = Object.keys(values as Record<string, unknown>)
        return keys.some(key => key !== 'true' && key !== 'false')
      })
      .map(([key]) => key)
    
    console.log('提取的变体列表:', variants)
    // ['color', 'variant', 'size'] (排除了 boolean 类型的变体如 loading, square 等)
    
    // 第四步：序列化主题对象
    let json = JSON.stringify(result, null, 2)
    console.log('序列化后的 JSON (部分):', json.substring(0, 500) + '...')
    
    // 第五步：添加类型声明替换
    for (const variant of variants) {
      // 为每个变体添加 TypeScript 类型声明
      json = json.replace(new RegExp(`("${variant}": "[^"]+")`, 'g'), `$1 as typeof ${variant}[number]`)
      json = json.replace(new RegExp(`("${variant}": \\[\\s*)((?:"[^"]+",?\\s*)+)(\\])`, 'g'), (_, before, match, after) => {
        const replaced = match.replace(/("[^"]+")/g, `$1 as typeof ${variant}[number]`)
        return `${before}${replaced}${after}`
      })
    }
    
    // 第六步：生成变体常量声明
    function generateVariantDeclarations(variants: string[]) {
      return variants.filter(variant => json.includes(`as typeof ${variant}`)).map((variant) => {
        const keys = Object.keys(result.variants[variant])
        return `const ${variant} = ${JSON.stringify(keys, null, 2)} as const`
      })
    }
    
    const variantDeclarations = generateVariantDeclarations(variants)
    console.log('生成的变体声明:', variantDeclarations)
    // [
    //   'const color = ["primary", "brand", "accent", "neutral"] as const',
    //   'const variant = ["solid", "outline", "soft", "subtle", "ghost", "link"] as const', 
    //   'const size = ["xs", "sm", "md", "lg", "xl"] as const'
    // ]
    
    // 第七步：组装最终的文件内容
    const finalContent = [
      ...variantDeclarations,
      `export default ${json}`
    ].join('\n\n')
    
    console.log('生成的最终文件内容预览:', finalContent.substring(0, 1000) + '...')
    
    return finalContent
  }
})
```

### 3.3 实际生成的文件内容

**生成的 `#build/ui/button.ts` 文件完整内容**
```typescript
const color = [
  "primary",
  "brand", 
  "accent",
  "neutral"
] as const

const variant = [
  "solid",
  "outline",
  "soft",
  "subtle", 
  "ghost",
  "link"
] as const

const size = [
  "xs",
  "sm",
  "md",
  "lg", 
  "xl"
] as const

export default {
  "slots": {
    "base": [
      "rounded-md font-medium inline-flex items-center disabled:cursor-not-allowed aria-disabled:cursor-not-allowed disabled:opacity-75 aria-disabled:opacity-75",
      "transition-colors"
    ],
    "label": "truncate",
    "leadingIcon": "shrink-0",
    "leadingAvatar": "shrink-0", 
    "leadingAvatarSize": "",
    "trailingIcon": "shrink-0"
  },
  "variants": {
    "buttonGroup": {
      "horizontal": "not-only:first:rounded-e-none not-only:last:rounded-s-none not-last:not-first:rounded-none focus-visible:z-[1]",
      "vertical": "not-only:first:rounded-b-none not-only:last:rounded-t-none not-last:not-first:rounded-none focus-visible:z-[1]"
    },
    "color": {
      "primary": "",
      "brand": "",
      "accent": "",
      "neutral": ""
    },
    "variant": {
      "solid": "",
      "outline": "",
      "soft": "",
      "subtle": "",
      "ghost": "",
      "link": ""
    },
    "size": {
      "xs": {
        "base": "px-2 py-1 text-xs gap-1",
        "leadingIcon": "size-4",
        "leadingAvatarSize": "3xs",
        "trailingIcon": "size-4"
      },
      "sm": {
        "base": "px-2.5 py-1.5 text-xs gap-1.5", 
        "leadingIcon": "size-4",
        "leadingAvatarSize": "3xs",
        "trailingIcon": "size-4"
      },
      "md": {
        "base": "px-2.5 py-1.5 text-sm gap-1.5",
        "leadingIcon": "size-5",
        "leadingAvatarSize": "2xs",
        "trailingIcon": "size-5"
      },
      "lg": {
        "base": "px-3 py-2 text-sm gap-2",
        "leadingIcon": "size-5", 
        "leadingAvatarSize": "2xs",
        "trailingIcon": "size-5"
      },
      "xl": {
        "base": "px-3 py-2 text-base gap-2",
        "leadingIcon": "size-6",
        "leadingAvatarSize": "xs",
        "trailingIcon": "size-6"
      }
    },
    "block": {
      "true": {
        "base": "w-full justify-center",
        "trailingIcon": "ms-auto"
      }
    },
    "square": {
      "true": ""
    },
    "leading": {
      "true": ""
    },
    "trailing": {
      "true": ""
    },
    "loading": {
      "true": ""
    },
    "active": {
      "true": {
        "base": ""
      },
      "false": {
        "base": ""
      }
    }
  },
  "compoundVariants": [
    {
      "color": "primary",
      "variant": "solid",
      "class": "text-inverted bg-primary hover:bg-primary/75 disabled:bg-primary aria-disabled:bg-primary focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-primary"
    },
    {
      "color": "brand", 
      "variant": "solid",
      "class": "text-inverted bg-brand hover:bg-brand/75 disabled:bg-brand aria-disabled:bg-brand focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-brand"
    },
    {
      "color": "accent",
      "variant": "solid", 
      "class": "text-inverted bg-accent hover:bg-accent/75 disabled:bg-accent aria-disabled:bg-accent focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-accent"
    },
    // ... 更多复合变体 (约30个)
  ],
  "defaultVariants": {
    "color": "primary",
    "variant": "solid",
    "size": "md"
  }
}
```

### 3.4 CSS 文件生成过程

**CSS 模板生成 (`templates.ts` 第 67 行)**
```typescript
templates.push({
  filename: 'ui.css',
  write: true,
  getContents: () => {
    console.log('开始生成 ui.css 文件')
    
    // 第一步：处理用户自定义颜色的 CSS 变量
    const customColors = [...(options.theme?.colors || []).filter(color => 
      !colors[color as keyof typeof colors]  // 排除 TailwindCSS 默认颜色
    ), 'neutral']
    
    console.log('需要生成 CSS 变量的颜色:', customColors)
    // ['brand', 'accent', 'neutral'] (primary 是 TailwindCSS 默认颜色，不需要)
    
    // 第二步：生成颜色变量映射
    const colorVariables = customColors.map(color => 
      [50, 100, 200, 300, 400, 500, 600, 700, 800, 900, 950].map(shade => 
        `--color-${color}-${shade}: var(--ui-color-${color}-${shade});`
      ).join('\n\t')
    ).join('\n\t')
    
    console.log('生成的颜色变量:', colorVariables)
    // --color-brand-50: var(--ui-color-brand-50);
    // --color-brand-100: var(--ui-color-brand-100);
    // ... 
    // --color-accent-50: var(--ui-color-accent-50);
    // ...
    // --color-neutral-50: var(--ui-color-neutral-50);
    // ...
    
    // 第三步：生成语义色彩变量
    const semanticColors = options.theme?.colors?.map(color => 
      `--color-${color}: var(--ui-${color});`
    ).join('\n\t')
    
    console.log('生成的语义色彩变量:', semanticColors)
    // --color-primary: var(--ui-primary);
    // --color-brand: var(--ui-brand);
    // --color-accent: var(--ui-accent);
    
    return `@source "./ui";

@theme static {
  --color-old-neutral-50: ${colors.neutral[50]};
  --color-old-neutral-100: ${colors.neutral[100]};
  --color-old-neutral-200: ${colors.neutral[200]};
  --color-old-neutral-300: ${colors.neutral[300]};
  --color-old-neutral-400: ${colors.neutral[400]};
  --color-old-neutral-500: ${colors.neutral[500]};
  --color-old-neutral-600: ${colors.neutral[600]};
  --color-old-neutral-700: ${colors.neutral[700]};
  --color-old-neutral-800: ${colors.neutral[800]};
  --color-old-neutral-900: ${colors.neutral[900]};
  --color-old-neutral-950: ${colors.neutral[950]};
}

@theme default inline {
  ${colorVariables}
  ${semanticColors}
  --radius-xs: calc(var(--ui-radius) * 0.5);
  --radius-sm: var(--ui-radius);
  --radius-md: calc(var(--ui-radius) * 1.5);
  --radius-lg: calc(var(--ui-radius) * 2);
  --radius-xl: calc(var(--ui-radius) * 3);
  --radius-2xl: calc(var(--ui-radius) * 4);
  --radius-3xl: calc(var(--ui-radius) * 6);
  --text-color-dimmed: var(--ui-text-dimmed);
  --text-color-muted: var(--ui-text-muted);
  --text-color-toned: var(--ui-text-toned);
  --text-color-default: var(--ui-text);
  --text-color-highlighted: var(--ui-text-highlighted);
  --text-color-inverted: var(--ui-text-inverted);
  --background-color-default: var(--ui-bg);
  --background-color-muted: var(--ui-bg-muted);
  --background-color-elevated: var(--ui-bg-elevated);
  --background-color-accented: var(--ui-bg-accented);
  --background-color-inverted: var(--ui-bg-inverted);
  --background-color-border: var(--ui-border);
  --border-color-default: var(--ui-border);
  --border-color-muted: var(--ui-border-muted);
  --border-color-accented: var(--ui-border-accented);
  --border-color-inverted: var(--ui-border-inverted);
  --border-color-bg: var(--ui-bg);
  --ring-color-default: var(--ui-border);
  --ring-color-muted: var(--ui-border-muted);
  --ring-color-accented: var(--ui-border-accented);
  --ring-color-inverted: var(--ui-border-inverted);
  --ring-color-bg: var(--ui-bg);
  --ring-offset-color-default: var(--ui-border);
  --ring-offset-color-muted: var(--ui-border-muted);
  --ring-offset-color-accented: var(--ui-border-accented);
  --ring-offset-color-inverted: var(--ui-border-inverted);
  --ring-offset-color-bg: var(--ui-bg);
  --divide-color-default: var(--ui-border);
  --divide-color-muted: var(--ui-border-muted);
  --divide-color-accented: var(--ui-border-accented);
  --divide-color-inverted: var(--ui-border-inverted);
  --divide-color-bg: var(--ui-bg);
  --outline-color-default: var(--ui-border);
  --outline-color-inverted: var(--ui-border-inverted);
  --stroke-default: var(--ui-border);
  --stroke-inverted: var(--ui-border-inverted);
  --fill-default: var(--ui-border);
  --fill-inverted: var(--ui-border-inverted);
}
`
  }
})
```

## 4. 运行时应用阶段

### 4.1 TV 实例创建过程 (`runtime/utils/tv.ts`)

**步骤 1: 导入依赖**
```typescript
import { createTV, type defaultConfig } from 'tailwind-variants'
import type { AppConfig } from '@nuxt/schema'
import appConfig from '#build/app.config'
```

**输入分析:**
- `appConfig` 来自构建时生成的 `#build/app.config.ts`
- 内容示例:
```typescript
// #build/app.config.ts 生成的内容
{
  ui: {
    tv: {
      // TV 全局配置
      prefix: '',
      separator: ':',
      responsivePrefix: '',
      responsiveOperator: 'max-width',
      // ... 其他配置
    },
    colors: {
      primary: 'blue',
      secondary: 'gray',
      success: 'green',
      // ...
    },
    button: {
      // 按钮特定配置
      variants: {
        customVariant: {
          special: 'bg-gradient-to-r from-blue-500 to-purple-600'
        }
      }
    }
  }
}
```

**步骤 2: TV 实例创建**
```typescript
const appConfigTv = appConfig as AppConfig & { ui: { tv: typeof defaultConfig } }
export const tv = /* @__PURE__ */ createTV(appConfigTv.ui?.tv)
```

**输出结果:**
- 返回一个全局配置的 TV 实例
- 这个实例包含所有的默认配置和用户自定义配置

### 4.2 Button 组件中的详细应用过程

**步骤 1: 导入和基础设置**
```vue
<script setup lang="ts">
import theme from '#build/ui/button'  // 构建时生成的主题
import { tv } from '../utils/tv'       // 全局 TV 实例
import { defu } from 'defu'           // 深度合并工具
import { computed } from 'vue'

// Props 定义示例
interface Props {
  color?: 'primary' | 'secondary' | 'success' | 'error' | string
  variant?: 'solid' | 'outline' | 'soft' | 'ghost' | string
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl'
  loading?: boolean
  block?: boolean
  ui?: Partial<ButtonUI>
  class?: string
  // ... 其他 props
}

const props = withDefaults(defineProps<Props>(), {
  color: 'primary',
  variant: 'solid',
  size: 'md'
})
</script>
```

**步骤 2: 主题配置计算 - 详细过程**

```typescript
const ui = computed(() => {
  // 输入 1: 基础主题 (来自 #build/ui/button.ts)
  const baseTheme = theme
  console.log('基础主题:', baseTheme)
  // 输出示例:
  // {
  //   slots: {
  //     base: ['rounded-md', 'font-medium', 'inline-flex', 'items-center', ...],
  //     leadingIcon: ['shrink-0'],
  //     trailingIcon: ['shrink-0']
  //   },
  //   variants: {
  //     color: { primary: '', secondary: '', ... },
  //     variant: { solid: '', outline: '', ... },
  //     size: { xs: { base: ['px-2', 'py-1', ...] }, ... }
  //   },
  //   compoundVariants: [
  //     { color: 'primary', variant: 'solid', class: 'text-white bg-primary-500 ...' },
  //     ...
  //   ],
  //   defaultVariants: { color: 'primary', variant: 'solid', size: 'md' }
  // }

  // 输入 2: AppConfig 中的按钮配置
  const appConfigButton = appConfig.ui?.button || {}
  console.log('AppConfig 按钮配置:', appConfigButton)
  // 输出示例:
  // {
  //   variants: {
  //     customVariant: {
  //       special: 'bg-gradient-to-r from-blue-500 to-purple-600'
  //     }
  //   }
  // }

  // 输入 3: 组件级自定义配置
  const componentUI = {
    variants: {
      active: {
        true: { base: props.activeClass || 'ring-2 ring-primary-500' },
        false: { base: props.inactiveClass || '' }
      }
    }
  }
  console.log('组件级配置:', componentUI)

  // 步骤 2.1: 创建扩展的 TV 实例
  const extendedTv = tv({
    extend: tv(baseTheme),  // 继承基础主题
    ...defu(componentUI, appConfigButton)  // 合并配置
  })
  console.log('扩展后的 TV 实例:', extendedTv)

  return extendedTv
})
```

**步骤 3: 样式计算 - 具体示例**

假设我们有一个按钮使用以下 props:
```vue
<UButton 
  color="primary" 
  variant="solid" 
  size="md" 
  loading="true"
  :ui="{ base: 'shadow-lg' }"
>
  Click me
</UButton>
```

**计算过程:**

```typescript
// 步骤 3.1: 收集所有变体值
const variantValues = {
  color: props.color,           // 'primary'
  variant: props.variant,       // 'solid'
  size: buttonSize.value,       // 'md'
  loading: isLoading.value,     // true
  block: props.block,           // undefined (false)
  square: props.square || (!slots.default && !props.label), // false
  leading: isLeading.value,     // false (没有 leading 内容)
  trailing: isTrailing.value    // false (没有 trailing 内容)
}
console.log('变体值:', variantValues)

// 步骤 3.2: TV 计算样式类名
const calculatedStyles = ui.value(variantValues)
console.log('计算出的样式:', calculatedStyles)

// 输出示例:
// {
//   base: () => 'rounded-md font-medium inline-flex items-center px-2.5 py-1.5 text-sm gap-1.5 text-white bg-primary-500 hover:bg-primary-600 focus:outline-none focus:ring-2 focus:ring-primary-500 focus:ring-offset-2 disabled:opacity-50',
//   leadingIcon: () => 'shrink-0 size-4',
//   trailingIcon: () => 'shrink-0 size-4',
//   label: () => 'truncate'
```

**步骤 4: 最终类名生成**

```typescript
// 在模板中使用
const finalClasses = ui.base({
  class: [
    props.ui?.base,    // 'shadow-lg' (来自 props)
    props.class        // 额外的类名
  ],
  active: isActive.value,  // false
  loading: isLoading.value // true
})

console.log('最终类名:', finalClasses)
// 输出: 'rounded-md font-medium inline-flex items-center px-2.5 py-1.5 text-sm gap-1.5 text-white bg-primary-500 hover:bg-primary-600 focus:outline-none focus:ring-2 focus:ring-primary-500 focus:ring-offset-2 disabled:opacity-50 shadow-lg'
```

### 4.3 完整的样式计算流程图

```
输入 Props:
├─ color: 'primary'
├─ variant: 'solid'  
├─ size: 'md'
├─ loading: true
└─ ui: { base: 'shadow-lg' }
         │
         ▼
基础主题解析 (theme/button.ts):
├─ slots.base: ['rounded-md', 'font-medium', 'inline-flex', 'items-center', ...]
├─ variants.size.md.base: ['px-2.5', 'py-1.5', 'text-sm', ...]
└─ compoundVariants 匹配: color='primary' + variant='solid'
         │
         ▼
AppConfig 配置合并:
├─ 自定义变体
├─ 颜色覆盖
└─ 全局 TV 配置
         │
         ▼
TV 实例计算:
├─ 匹配基础样式
├─ 匹配变体样式
├─ 匹配复合变体
├─ 应用插槽样式
└─ 合并自定义类名
         │
         ▼
最终输出:
'rounded-md font-medium inline-flex items-center px-2.5 py-1.5 text-sm gap-1.5 text-white bg-primary-500 hover:bg-primary-600 focus:outline-none focus:ring-2 focus:ring-primary-500 focus:ring-offset-2 disabled:opacity-50 shadow-lg'
```

### 4.4 模板中的实际应用

```vue
<template>
  <ULinkBase
    :class="ui.base({
      class: [props.ui?.base, props.class],
      active: isActive.value,
      loading: isLoading.value
    })"
  >
    <!-- Leading Icon 示例 -->
    <UIcon 
      v-if="isLeading && leadingIconName" 
      :name="leadingIconName"
      :class="ui.leadingIcon({ 
        class: props.ui?.leadingIcon,
        active: isActive.value
      })" 
    />
    <!-- 计算出的类名: 'shrink-0 size-4' -->
    
    <!-- Label 示例 -->
    <span 
      v-if="props.label"
      :class="ui.label({ 
        class: props.ui?.label 
      })"
    >
      {{ props.label }}
    </span>
    <!-- 计算出的类名: 'truncate' -->
    
    <!-- Trailing Icon 示例 -->
    <UIcon 
      v-if="isTrailing && trailingIconName" 
      :name="trailingIconName"
      :class="ui.trailingIcon({ 
        class: props.ui?.trailingIcon,
        active: isActive.value
      })" 
    />
    <!-- 计算出的类名: 'shrink-0 size-4' -->
  </ULinkBase>
</template>
```

## 5. TV 工作原理 - 具体计算示例

### 5.1 变体匹配算法详解

**输入示例:**
```typescript
const input = {
  color: 'primary',
  variant: 'solid',
  size: 'md',
  loading: true
}
```

**步骤 1: 基础样式收集**
```typescript
// 从 slots.base 获取
const baseClasses = ['rounded-md', 'font-medium', 'inline-flex', 'items-center', 'justify-center', 'gap-1', 'focus:outline-none', 'disabled:cursor-not-allowed', 'disabled:opacity-75']
console.log('基础样式:', baseClasses)
```

**步骤 2: 单一变体匹配**
```typescript
// size: 'md' 匹配
const sizeClasses = {
  base: ['px-2.5', 'py-1.5', 'text-sm', 'gap-1.5'],
  leadingIcon: ['size-4'],
  trailingIcon: ['size-4']
}
console.log('尺寸样式:', sizeClasses)

// loading: true 匹配
const loadingClasses = {
  base: ['cursor-default']
}
console.log('加载样式:', loadingClasses)
```

**步骤 3: 复合变体匹配**
```typescript
// color: 'primary' + variant: 'solid' 匹配
const compoundClasses = {
  base: ['text-white', 'bg-primary-500', 'hover:bg-primary-600', 'focus:ring-2', 'focus:ring-primary-500', 'focus:ring-offset-2']
}
console.log('复合变体样式:', compoundClasses)
```

**步骤 4: 样式合并**
```typescript
const finalBaseClasses = [
  ...baseClasses,        // 基础样式
  ...sizeClasses.base,   // 尺寸样式
  ...loadingClasses.base, // 加载样式
  ...compoundClasses.base // 复合变体样式
]
console.log('最终基础样式:', finalBaseClasses.join(' '))
// 输出: 'rounded-md font-medium inline-flex items-center justify-center gap-1 focus:outline-none disabled:cursor-not-allowed disabled:opacity-75 px-2.5 py-1.5 text-sm gap-1.5 cursor-default text-white bg-primary-500 hover:bg-primary-600 focus:ring-2 focus:ring-primary-500 focus:ring-offset-2'
```

### 5.2 插槽样式计算示例

```typescript
// leadingIcon 插槽计算
const leadingIconClasses = [
  'shrink-0',                    // 从 slots.leadingIcon
  ...sizeClasses.leadingIcon     // 从 size 变体: ['size-4']
]
console.log('Leading Icon 样式:', leadingIconClasses.join(' '))
// 输出: 'shrink-0 size-4'

// trailingIcon 插槽计算  
const trailingIconClasses = [
  'shrink-0',                    // 从 slots.trailingIcon
  ...sizeClasses.trailingIcon    // 从 size 变体: ['size-4']
]
console.log('Trailing Icon 样式:', trailingIconClasses.join(' '))
// 输出: 'shrink-0 size-4'
```

### 5.3 自定义类名合并示例

```typescript
// 组件使用示例
<UButton 
  color="primary"
  variant="solid" 
  size="md"
  :ui="{ 
    base: 'shadow-lg transform hover:scale-105',
    leadingIcon: 'text-yellow-400'
  }"
  class="my-custom-class"
>

// 最终类名计算
const finalClasses = ui.base({
  class: [
    'shadow-lg transform hover:scale-105', // props.ui.base
    'my-custom-class'                      // props.class
  ]
})

console.log('最终类名:', finalClasses)
// 输出: 'rounded-md font-medium inline-flex items-center justify-center gap-1 focus:outline-none disabled:cursor-not-allowed disabled:opacity-75 px-2.5 py-1.5 text-sm gap-1.5 text-white bg-primary-500 hover:bg-primary-600 focus:ring-2 focus:ring-primary-500 focus:ring-offset-2 shadow-lg transform hover:scale-105 my-custom-class'
```

## 6. 扩展和自定义机制 - 详细实例分析

### 6.1 AppConfig 层面的自定义

**场景: 修改默认颜色配置**

**输入 (`app.config.ts`):**
```typescript
export default defineAppConfig({
  ui: {
    colors: {
      primary: 'blue',     // 默认
      secondary: 'green',  // 覆盖默认的 'gray'
      custom: 'purple'     // 新增自定义颜色
    },
    button: {
      variants: {
        size: {
          '2xl': {
            base: ['px-6', 'py-4', 'text-xl', 'gap-3'],
            leadingIcon: ['size-6'],
            trailingIcon: ['size-6']
          }
        },
        theme: {
          neon: {
            base: ['border-2', 'border-cyan-400', 'shadow-cyan-400/50', 'shadow-lg']
          }
        }
      }
    }
  }
})
```

**处理过程:**

1. **颜色解析阶段** (`module.ts` 中的 `resolveColors`):
```typescript
// 输入: 用户配置的颜色
const userColors = { primary: 'blue', secondary: 'green', custom: 'purple' }

// 解析后的颜色配置
const resolvedColors = {
  primary: {
    50: '#eff6ff',
    100: '#dbeafe', 
    // ... 完整的颜色谱
    900: '#1e3a8a',
    950: '#1e40af'
  },
  secondary: {
    50: '#f0fdf4',
    100: '#dcfce7',
    // ... green 颜色谱
    900: '#14532d',
    950: '#052e16'
  },
  custom: {
    50: '#faf5ff',
    100: '#f3e8ff',
    // ... purple 颜色谱  
    900: '#581c87',
    950: '#3b0764'
  }
}
console.log('解析后的颜色:', resolvedColors)
```

2. **主题生成阶段** (构建时):
```typescript
// Button 主题生成器会接收到扩展的颜色配置
const extendedTheme = createButtonTheme({
  colors: resolvedColors,
  customVariants: {
    size: {
      '2xl': {
        base: ['px-6', 'py-4', 'text-xl', 'gap-3'],
        leadingIcon: ['size-6'],
        trailingIcon: ['size-6']
      }
    },
    theme: {
      neon: {
        base: ['border-2', 'border-cyan-400', 'shadow-cyan-400/50', 'shadow-lg']
      }
    }
  }
})

console.log('扩展后的主题变体:', extendedTheme.variants)
// 输出包含新的 size='2xl' 和 theme='neon' 变体
```

**使用效果:**
```vue
<!-- 现在可以使用新的配置 -->
<UButton color="custom" size="2xl" theme="neon">
  大尺寸紫色霓虹按钮
</UButton>

<!-- 最终生成的类名 -->
<!-- 
  rounded-md font-medium inline-flex items-center justify-center gap-1 
  focus:outline-none disabled:cursor-not-allowed disabled:opacity-75 
  px-6 py-4 text-xl gap-3 
  border-2 border-cyan-400 shadow-cyan-400/50 shadow-lg 
  text-white bg-custom-500 hover:bg-custom-600 
  focus:ring-2 focus:ring-custom-500 focus:ring-offset-2
-->
```

### 6.2 组件级自定义 - 深度定制示例

**场景: 创建一个带动画的自定义按钮**

```vue
<template>
  <UButton
    v-bind="$attrs"
    :color="color"
    :variant="variant"
    :size="size"
    :ui="customUI"
    :class="animationClass"
    @click="handleClick"
  >
    <template #leading>
      <UIcon 
        v-if="loading" 
        name="i-lucide-loader-2" 
        class="animate-spin" 
      />
      <slot v-else name="leading" />
    </template>
    
    <slot />
    
    <template #trailing>
      <slot name="trailing" />
    </template>
  </UButton>
</template>

<script setup lang="ts">
interface Props {
  color?: string
  variant?: string
  size?: string
  loading?: boolean
  animationType?: 'bounce' | 'pulse' | 'wiggle'
}

const props = withDefaults(defineProps<Props>(), {
  color: 'primary',
  variant: 'solid',
  size: 'md',
  animationType: 'bounce'
})

// 自定义 UI 配置 - 详细分析
const customUI = computed(() => {
  return {
    base: [
      // 基础动画类
      'transition-all duration-200 ease-in-out',
      // 根据动画类型添加不同的效果
      props.animationType === 'bounce' ? 'hover:scale-105 active:scale-95' : '',
      props.animationType === 'pulse' ? 'hover:animate-pulse' : '',
      props.animationType === 'wiggle' ? 'hover:animate-wiggle' : '',
      // 渐变背景效果
      'bg-gradient-to-r hover:bg-gradient-to-l',
      // 阴影效果
      'shadow-md hover:shadow-lg',
      // 加载状态下的样式
      props.loading ? 'cursor-wait opacity-75' : ''
    ].filter(Boolean).join(' '),
    
    leadingIcon: [
      'transition-transform duration-200',
      props.loading ? 'animate-spin' : 'group-hover:rotate-12'
    ].filter(Boolean).join(' '),
    
    trailingIcon: [
      'transition-all duration-200',
      'group-hover:translate-x-1'
    ].join(' ')
  }
})

// 动态类名计算
const animationClass = computed(() => {
  const classes = ['group'] // 用于子元素的 group-hover 效果
  
  if (props.loading) {
    classes.push('animate-pulse')
  }
  
  return classes.join(' ')
})

const handleClick = (event: Event) => {
  if (props.loading) {
    event.preventDefault()
    return
  }
  
  // 触发点击动画
  const button = event.target as HTMLElement
  button.classList.add('animate-bounce')
  setTimeout(() => {
    button.classList.remove('animate-bounce')
  }, 600)
}
</script>
```

**最终生成的类名分析:**

```typescript
// 当使用 <CustomButton color="primary" variant="solid" size="md" animationType="bounce">
// 最终的 base 类名将是:

const finalClasses = [
  // 来自 Button 基础主题
  'rounded-md', 'font-medium', 'inline-flex', 'items-center', 'justify-center',
  'gap-1', 'focus:outline-none', 'disabled:cursor-not-allowed', 'disabled:opacity-75',
  
  // 来自 size="md" 变体
  'px-2.5', 'py-1.5', 'text-sm', 'gap-1.5',
  
  // 来自 color="primary" + variant="solid" 复合变体
  'text-white', 'bg-primary-500', 'hover:bg-primary-600',
  'focus:ring-2', 'focus:ring-primary-500', 'focus:ring-offset-2',
  
  // 来自自定义 UI 配置
  'transition-all', 'duration-200', 'ease-in-out',
  'hover:scale-105', 'active:scale-95',
  'bg-gradient-to-r', 'hover:bg-gradient-to-l',
  'shadow-md', 'hover:shadow-lg',
  
  // 来自组件类名
  'group'
].join(' ')

console.log('最终类名:', finalClasses)
```

### 6.3 主题扩展机制 - TV 继承和覆盖

**场景: 创建一个专门的 CTA 按钮主题**

```typescript
// composables/useCTAButton.ts
export const useCTAButton = () => {
  // 获取基础 Button 主题
  const baseButtonTheme = theme // 从 #build/ui/button 导入

  // 创建扩展的 CTA 按钮主题
  const ctaButtonTheme = tv({
    extend: tv(baseButtonTheme), // 继承基础按钮主题
    
    // 覆盖和扩展配置
    slots: {
      base: [
        // 保留基础样式，添加 CTA 特定样式
        'relative overflow-hidden',
        'before:absolute before:inset-0',
        'before:bg-gradient-to-r before:from-transparent before:via-white/20 before:to-transparent',
        'before:translate-x-[-100%] hover:before:translate-x-[100%]',
        'before:transition-transform before:duration-700 before:ease-out'
      ],
      // 添加新的插槽
      shine: ['absolute inset-0 bg-gradient-to-r from-transparent via-white/10 to-transparent']
    },
    
    variants: {
      // 扩展尺寸变体
      size: {
        cta: {
          base: ['px-8', 'py-4', 'text-lg', 'font-bold', 'min-w-[200px]'],
          leadingIcon: ['size-6'],
          trailingIcon: ['size-6']
        }
      },
      
      // 新增 CTA 特定变体
      ctaStyle: {
        primary: {
          base: [
            'bg-gradient-to-r from-blue-600 to-purple-600',
            'hover:from-blue-700 hover:to-purple-700',
            'shadow-lg shadow-blue-500/25'
          ]
        },
        success: {
          base: [
            'bg-gradient-to-r from-green-500 to-emerald-600',
            'hover:from-green-600 hover:to-emerald-700',
            'shadow-lg shadow-green-500/25'
          ]
        },
        warning: {
          base: [
            'bg-gradient-to-r from-orange-500 to-red-500',
            'hover:from-orange-600 hover:to-red-600',
            'shadow-lg shadow-orange-500/25'
          ]
        }
      },
      
      // 闪光效果变体
      shine: {
        true: {
          base: ['animate-shine']
        }
      }
    },
    
    // 新的复合变体
    compoundVariants: [
      {
        ctaStyle: 'primary',
        shine: true,
        class: 'animate-pulse-glow'
      },
      {
        size: 'cta',
        ctaStyle: ['primary', 'success', 'warning'],
        class: 'transform hover:scale-105 active:scale-95 transition-transform duration-200'
      }
    ],
    
    // 默认变体
    defaultVariants: {
      size: 'cta',
      ctaStyle: 'primary',
      shine: false
    }
  })

  return { ctaButtonTheme }
}
```

**使用示例和输出分析:**

```vue
<script setup>
const { ctaButtonTheme } = useCTAButton()

// 计算 CTA 按钮样式
const ctaStyles = computed(() => {
  return ctaButtonTheme({
    size: 'cta',
    ctaStyle: 'primary', 
    shine: true
  })
})
</script>

<template>
  <button :class="ctaStyles.base()">
    <span :class="ctaStyles.shine()"></span>
    立即购买
  </button>
</template>
```

**输出的类名分析:**

```typescript
// base 插槽的最终类名:
const ctaBaseClasses = [
  // 继承自基础 Button 主题
  'rounded-md', 'font-medium', 'inline-flex', 'items-center', 'justify-center',
  'gap-1', 'focus:outline-none', 'disabled:cursor-not-allowed', 'disabled:opacity-75',
  
  // 来自 size="cta" 变体
  'px-8', 'py-4', 'text-lg', 'font-bold', 'min-w-[200px]',
  
  // 来自 ctaStyle="primary" 变体
  'bg-gradient-to-r', 'from-blue-600', 'to-purple-600',
  'hover:from-blue-700', 'hover:to-purple-700',
  'shadow-lg', 'shadow-blue-500/25',
  
  // 来自扩展的 base 插槽
  'relative', 'overflow-hidden',
  'before:absolute', 'before:inset-0',
  'before:bg-gradient-to-r', 'before:from-transparent', 'before:via-white/20', 'before:to-transparent',
  'before:translate-x-[-100%]', 'hover:before:translate-x-[100%]',
  'before:transition-transform', 'before:duration-700', 'before:ease-out',
  
  // 来自 shine=true 变体
  'animate-shine',
  
  // 来自复合变体
  'animate-pulse-glow',
  'transform', 'hover:scale-105', 'active:scale-95', 'transition-transform', 'duration-200'
].join(' ')

console.log('CTA 按钮最终类名:', ctaBaseClasses)
```

## 7. 实际应用示例 - 完整数据流

### 7.1 复杂按钮组合示例

**场景: 创建一个带图标、加载状态、自定义样式的提交按钮**

```vue
<template>
  <div class="space-y-4">
    <!-- 示例 1: 基础提交按钮 -->
    <UButton
      color="primary"
      variant="solid"
      size="lg"
      :loading="isSubmitting"
      :disabled="!isFormValid"
      @click="handleSubmit"
    >
      <template #leading>
        <UIcon name="i-lucide-send" />
      </template>
      {{ isSubmitting ? '提交中...' : '提交表单' }}
    </UButton>

    <!-- 示例 2: 带自定义 UI 的危险操作按钮 -->
    <UButton
      color="red"
      variant="outline"
      size="md"
      :ui="dangerButtonUI"
      @click="handleDelete"
    >
      <template #leading>
        <UIcon name="i-lucide-trash-2" />
      </template>
      删除数据
    </UButton>

    <!-- 示例 3: 社交登录按钮 -->
    <UButton
      variant="outline"
      size="lg"
      :ui="socialButtonUI"
      class="w-full"
      @click="handleGithubLogin"
    >
      <template #leading>
        <UIcon name="i-lucide-github" />
      </template>
      使用 GitHub 登录
    </UButton>
  </div>
</template>

<script setup lang="ts">
const isSubmitting = ref(false)
const isFormValid = ref(true)

// 危险操作按钮的自定义 UI
const dangerButtonUI = {
  base: [
    'hover:bg-red-50 focus:bg-red-50',
    'border-red-300 hover:border-red-400',
    'text-red-700 hover:text-red-800',
    'transition-all duration-200',
    'shadow-sm hover:shadow-md'
  ].join(' '),
  leadingIcon: 'text-red-600'
}

// 社交登录按钮的自定义 UI
const socialButtonUI = {
  base: [
    'bg-gray-900 hover:bg-gray-800',
    'border-gray-700 hover:border-gray-600',
    'text-white',
    'shadow-lg hover:shadow-xl',
    'transform hover:scale-[1.02]',
    'transition-all duration-200'
  ].join(' '),
  leadingIcon: 'text-white'
}

// 处理函数
const handleSubmit = async () => {
  isSubmitting.value = true
  try {
    // 模拟提交过程
    await new Promise(resolve => setTimeout(resolve, 2000))
    console.log('表单提交成功')
  } finally {
    isSubmitting.value = false
  }
}

const handleDelete = () => {
  if (confirm('确定要删除这些数据吗？')) {
    console.log('执行删除操作')
  }
}

const handleGithubLogin = () => {
  console.log('跳转到 GitHub OAuth')
}
</script>
```

**详细的类名生成过程:**

**示例 1 - 提交按钮 (loading 状态):**
```typescript
// 输入变体
const variants = {
  color: 'primary',
  variant: 'solid', 
  size: 'lg',
  loading: true,
  disabled: false,
  leading: true // 有 leading 插槽内容
}

// 计算过程
const styles = ui.value(variants)

// 最终的 base 类名:
const submitButtonClasses = [
  // 基础样式
  'rounded-md', 'font-medium', 'inline-flex', 'items-center', 'justify-center',
  'gap-1', 'focus:outline-none', 'disabled:cursor-not-allowed', 'disabled:opacity-75',
  
  // size="lg" 样式
  'px-3.5', 'py-2.5', 'text-base', 'gap-2',
  
  // color="primary" + variant="solid" 复合样式
  'text-white', 'bg-primary-500', 'hover:bg-primary-600',
  'focus:ring-2', 'focus:ring-primary-500', 'focus:ring-offset-2',
  
  // loading=true 样式
  'cursor-default'
].join(' ')

// leadingIcon 类名:
const leadingIconClasses = ['shrink-0', 'size-5'] // size-5 来自 lg 尺寸

console.log('提交按钮样式:', {
  base: submitButtonClasses,
  leadingIcon: leadingIconClasses.join(' ')
})
```

**示例 2 - 危险操作按钮:**
```typescript
// 输入变体
const variants = {
  color: 'red',
  variant: 'outline',
  size: 'md',
  leading: true
}

// 基础计算
const baseStyles = ui.value(variants)

// 合并自定义 UI
const finalStyles = {
  base: baseStyles.base({
    class: dangerButtonUI.base
  }),
  leadingIcon: baseStyles.leadingIcon({
    class: dangerButtonUI.leadingIcon
  })
}

// 最终类名:
const dangerButtonClasses = [
  // 基础样式
  'rounded-md', 'font-medium', 'inline-flex', 'items-center', 'justify-center',
  'gap-1', 'focus:outline-none', 'disabled:cursor-not-allowed', 'disabled:opacity-75',
  
  // size="md" 样式
  'px-2.5', 'py-1.5', 'text-sm', 'gap-1.5',
  
  // color="red" + variant="outline" 复合样式
  'text-red-700', 'border', 'border-red-200', 'hover:bg-red-50',
  'focus:ring-2', 'focus:ring-red-500', 'focus:ring-offset-2',
  
  // 自定义 UI 样式
  'hover:bg-red-50', 'focus:bg-red-50',
  'border-red-300', 'hover:border-red-400',
  'text-red-700', 'hover:text-red-800',
  'transition-all', 'duration-200',
  'shadow-sm', 'hover:shadow-md'
].join(' ')

console.log('危险操作按钮样式:', dangerButtonClasses)
```

### 7.2 性能优化分析

**TV 计算的性能特点:**

1. **缓存机制**:
```typescript
// TV 内部会缓存计算结果
const memoizedStyles = useMemo(() => {
  return ui.value({
    color: props.color,
    variant: props.variant,
    size: props.size
  })
}, [props.color, props.variant, props.size])
```

2. **构建时预计算**:
```typescript
// 主题在构建时已经生成，运行时只需要匹配和合并
const preComputedTheme = {
  // 所有可能的变体组合都已经预先计算
  variants: {
    // ... 完整的变体定义
  },
  compoundVariants: [
    // ... 所有复合变体组合
  ]
}
```

3. **按需生成**:
```typescript
// 只有使用到的样式才会被包含在最终的 CSS 中
// Tailwind CSS 的 purge 机制会移除未使用的样式
```

## 8. 总结：Nuxt UI Button TV 系统完整工作流程

### 8.1 系统架构概览

```
开发时配置 (app.config.ts)
         │
         ▼
模块初始化 (module.ts)
├─ 颜色解析和生成
├─ 用户配置合并
└─ AppConfig 注入
         │
         ▼
主题生成 (theme/button.ts)
├─ 基础样式定义
├─ 变体样式生成
├─ 复合变体计算
└─ 插槽样式配置
         │
         ▼
构建时处理 (templates.ts)
├─ 主题模板生成
├─ CSS 变量注入
└─ 类型定义生成
         │
         ▼
运行时应用 (Button.vue)
├─ TV 实例创建
├─ 变体值计算
├─ 样式类名生成
└─ DOM 渲染
```

### 8.2 关键设计模式

1. **分层配置系统**:
   - 默认配置 → AppConfig → 组件 Props → UI Props
   - 每一层都可以覆盖前一层的配置

2. **构建时优化**:
   - 主题在构建时预先生成
   - 避免运行时的重复计算
   - 支持 Tree-shaking 和代码分割

3. **类型安全**:
   - 完整的 TypeScript 支持
   - 编译时的变体验证
   - 智能代码提示

4. **可扩展性**:
   - 插件式的主题扩展
   - 组件级别的深度定制
   - 与设计系统的无缝集成

### 8.3 系统优势

1. **开发者体验**:
   - 直观的配置方式
   - 强大的自定义能力
   - 完整的类型支持

2. **性能表现**:
   - 构建时预计算
   - 运行时缓存机制
   - 按需加载和 Tree-shaking

3. **维护性**:
   - 清晰的架构分层
   - 标准化的扩展模式
   - 完善的文档和示例

4. **一致性**:
   - 统一的主题系统
   - 标准化的组件接口
   - 可预测的样式行为

这个详细的分析展示了 Nuxt UI 的 Button TV 系统如何通过精心设计的架构，实现了高性能、高度可定制且类型安全的组件样式管理方案。每个环节都有明确的职责和优化策略，形成了一个完整且强大的主题系统。
