# 初始化demo

src/components/monaco-editor/index.vue
```vue
<template>
<div ref="codeEditBox" class="codeEditBox"></div>
</template>

<script setup lang="ts">
import { defineComponent, onBeforeUnmount, onMounted, ref, watch, ComponentInternalInstance, getCurrentInstance, Ref } from 'vue'
import { editorProps } from './types/monaco-editor-type'
import jsonWorker from 'monaco-editor/esm/vs/language/json/json.worker?worker'
import cssWorker from 'monaco-editor/esm/vs/language/css/css.worker?worker'
import htmlWorker from 'monaco-editor/esm/vs/language/html/html.worker?worker'
import tsWorker from 'monaco-editor/esm/vs/language/typescript/ts.worker?worker'
import EditorWorker from 'monaco-editor/esm/vs/editor/editor.worker?worker'
import * as monaco from 'monaco-editor'

const editorOptions : Ref<editorProps> = ref({
modelValue: null,
width: '100%',
height: '40vh',
language: 'xml',
theme: 'OneDarkPro',
options: {
  automaticLayout: true,
	foldingStrategy: 'indentation',
	renderLineHighlight: 'all',
	selectOnLineNumbers: true,
	minimap: {
	  enabled: true,
	},
	readOnly: false,
	fontSize: 16,
	scrollBeyondLastLine: false,
	overviewRulerBorder: false,
}
}) as Ref<editorProps>
const emit = defineEmits(['update:modelValue', 'change', 'editor-mounted'])
self.MonacoEnvironment = {
	getWorker(_: string, label: string) {
	  if (label === 'json') {
		return new jsonWorker()
	  }
	  if (['css', 'scss', 'less'].includes(label)) {
		return new cssWorker()
	  }
	  if (['html', 'handlebars', 'razor'].includes(label)) {
		return new htmlWorker()
	  }
	  if (['typescript', 'javascript'].includes(label)) {
		return new tsWorker()
	  }
	  return new EditorWorker()
	},
  }
  let editor: monaco.editor.IStandaloneCodeEditor
  const codeEditBox = ref()

  const init = () => {
	monaco.languages.typescript.javascriptDefaults.setDiagnosticsOptions({
	  noSemanticValidation: true,
	  noSyntaxValidation: false,
	})
	monaco.languages.typescript.javascriptDefaults.setCompilerOptions({
	  target: monaco.languages.typescript.ScriptTarget.ES2020,
	  allowNonTsExtensions: true,
	})

	editor = monaco.editor.create(codeEditBox.value, {
	  value: editorOptions.value.modelValue,
	  language: editorOptions.value.language,
	  theme: editorOptions.value.theme,
	  ...editorOptions.value.options,
	})

	// 监听值的变化
	editor.onDidChangeModelContent(() => {
	  const value = editor.getValue() //给父组件实时返回最新文本
	  emit('update:modelValue', value)
	  emit('change', value)
	})

	emit('editor-mounted', editor)
  }
  watch(
	() => editorOptions.value.modelValue,
	newValue => {
	  if (editor) {
		const value = editor.getValue()
		if (newValue !== value) {
		  editor.setValue(newValue)
		}
	  }
	}
  )

  watch(
	() => editorOptions.value.options,
	newValue => {
	  editor.updateOptions(newValue)
	},
	{ deep: true }
  )

  watch(
	() => editorOptions.value.language,
	newValue => {
	  monaco.editor.setModelLanguage(editor.getModel()!, newValue)
	}
  )

  onBeforeUnmount(() => {
	editor.dispose()
  })

  onMounted(() => {
	init()
  })
</script>

<style lang="scss" scoped>
.codeEditBox {
width: v-bind('editorOptions.width');
height: v-bind('editorOptions.height');
}
</style>
```

src/components/monaco-editor/types/monaco-editor-type.ts

```typescript
import { PropType } from 'vue'
   
export type Theme = 'vs' | 'hc-black' | 'vs-dark'
export type FoldingStrategy = 'auto' | 'indentation'
export type RenderLineHighlight = 'all' | 'line' | 'none' | 'gutter'
export interface Options {
  automaticLayout: boolean // 自适应布局
  foldingStrategy: FoldingStrategy // 折叠方式  auto | indentation
  renderLineHighlight: RenderLineHighlight // 行亮
  selectOnLineNumbers: boolean // 显示行号
  minimap: {
    // 关闭小地图
    enabled: boolean
  }
  readOnly: boolean // 只读
  fontSize: number // 字体大小
  scrollBeyondLastLine: boolean // 取消代码后面一大段空白
  overviewRulerBorder: boolean // 不要滚动条的边框
}
export const editorProps = {
  modelValue: {
    type: String as PropType<string>,
    default: null,
  },
  width: {
    type: [String, Number] as PropType<string | number>,
    default: '100%',
  },
  height: {
    type: [String, Number] as PropType<string | number>,
    default: '100%',
  },
  language: {
    type: String as PropType<string>,
    default: 'javascript',
  },
  theme: {
    type: String as PropType<Theme>,
    validator(value: string): boolean {
      return ['vs', 'hc-black', 'vs-dark'].includes(value)
    },
    default: 'vs-dark',
  },
  options: {
    type: Object as PropType<Options>,
    default: function () {
      return {
        automaticLayout: true,
        foldingStrategy: 'indentation',
        renderLineHighlight: 'all',
        selectOnLineNumbers: true,
        minimap: {
          enabled: true,
        },
        readOnly: false,
        fontSize: 16,
        scrollBeyondLastLine: false,
        overviewRulerBorder: false,
      }
    },
  },
}
```

vite.config.ts
```typescript
/**
 * https://vitejs.dev/config/
 */
import { defineConfig } from 'vite'
import plugins from './vite-plugins'
import path from 'path'
export default defineConfig({
  plugins,
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@images': path.resolve(__dirname, './src/assets/images'),
      '@types': path.resolve(__dirname, './src/@types'),
      '@models': path.resolve(__dirname, './src/@models'),
    },
  },
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `
          @import '@/scss/vars.scss';
          @import '@/scss/base.scss';
        `,
      }
    },
  },
  server: {
    open: true,
    host: true,
  },
  // 强制预构建插件包
  optimizeDeps: {
    include: [
      `monaco-editor/esm/vs/language/json/json.worker`,
      `monaco-editor/esm/vs/language/css/css.worker`,
      `monaco-editor/esm/vs/language/html/html.worker`,
      `monaco-editor/esm/vs/language/typescript/ts.worker`,
      `monaco-editor/esm/vs/editor/editor.worker`
    ], 
  },
})
```

1. 这里我们在vite里面引入了worker文件，内部worker用于不同语言项的智能提示内容，然后把对应的worker绑定到MonacoEnvironment对象中的getWorker方法里，当调用对应语言项时返回对应Worker内容
2. 在初始化过程中，调用`monaco.editor.create` 方法生成editor实例，传入绑定id，以及值/绑定文件语言/主题等配置即可生成一个editor多语言编辑器。
3. 相关常用editor API
```typescript
const editor = monaco.editor.create(codeEditBox.value, {
  value: props.modelValue,
  language: props.language,
  theme: props.theme,
  automaticLayout: true,
  ...props.options,
})
editor.setValue(val) // editor设置值
cosnt value = editor.getValue() // value 获取值
editor.updateOptions(newOptions) // 更新editor配置项
monaco.editor.setModelLanguage(editor.getModel()!, newLanguage) // 设置editor的语言
editor.dispose() // 销毁editor实例
```
4. 调用``onDidChangeModelContent`` 回调
		在用户触发editor文本内容变更时的回调，此时可以向外部组件更新文本内容

# 重构组件，将UI与model分开

```typescript
import * as monaco from 'monaco-editor'
import prettier from 'prettier/standalone'
import prettierHtml from 'prettier/parser-html'
import EditorWorker from 'monaco-editor/esm/vs/editor/editor.worker?worker'

export default class CodeEditor {
  private tag = '[Editor]'
  private model: monaco.editor.ITextModel
  private editor: monaco.editor.IStandaloneCodeEditor

  constructor(selector: string, contentChangeCallback: (content: string) => void, isDarkTheme: boolean) {
    this.initWorker()
    const dom = document.querySelector(selector)
    if (!dom) {
      throw new Error('invalid dom')
    }
    this.model = monaco.editor.createModel('', 'xml')
    const editorOptions: Options = {
      automaticLayout: true,
      foldingStrategy: 'indentation',
      renderLineHighlight: 'all',
      selectOnLineNumbers: true,
      minimap: {
        enabled: false,
      },
      readOnly: false,
      fontSize: 12,
      scrollBeyondLastLine: false,
      overviewRulerBorder: false,
    }
    this.model.onDidChangeContent(() => contentChangeCallback(this.getMinifiedValue()))
    this.editor = monaco.editor.create(dom as HTMLElement, { model: this.model, ...editorOptions })
    monaco.editor.defineTheme('dark', {
      base: 'vs-dark',
      inherit: true,
      colors: {
        'editor.background': '#1c1e2d',
        // 'editor.foreground': '',
        // 'editor.selectionBackground': '',
        // 'editor.lineHighlightBackground': '',
        // 'editorCursor.foreground': '',
        // 'editorWhitespace.foreground': '',
        // 'editorIndentGuide.activeBackground': '',
        // 'editorLineNumber.foreground': '',
        // 'editor.selectionHighlightBorder': '',
        // 'editorGutter.background': '',
        // 'scrollbarSlider.shadow': '',
        'scrollbarSlider.background': '#393c4e',
        // 'scrollbarSlider.activeBackground': '',
        // 'scrollbarSlider.hoverBackground': '',
      },
      rules: [],
    })
    monaco.editor.defineTheme('light', {
      base: 'vs',
      inherit: true,
      colors: {
        'editor.background': '#f6f8fc',
      },
      rules: [],
    })
    const themeName = isDarkTheme ? 'dark' : 'light'
    this.setTheme(themeName)
  }
  setTheme(themeName: string) {
    monaco.editor.setTheme(themeName)
  }

  initWorker() {
    self.MonacoEnvironment = {
      getWorker() {
        return new EditorWorker()
      },
    }
  }

  getValue() {
    return this.model.getValue()
  }

  getMinifiedValue() {
    const value = this.model.getValue()
    return value.replace(/(\n|\s\s)/g, '')
  }

  setValue(value: string) {
    this.model.setValue(value)
  }

  disable() {
    this.editor.updateOptions({ readOnly: true })
  }
  enable() {
    this.editor.updateOptions({ readOnly: false })
  }

  // 插入
  insertValue(text: string) {
    const position = this.editor.getPosition()
    if (!position) {
      throw new Error(`${this.tag} wrong position`)
    }

    const { lineNumber, column } = position

    const value = this.getValue()
    const splitedValue = value.split('\n')
    const lineValue = splitedValue[lineNumber - 1]

    splitedValue[lineNumber - 1] = [lineValue.slice(0, column - 1), text, lineValue.slice(column - 1)].join('')

    this.setValue(splitedValue.join('\n'))
    this.editor.setPosition(position)
  }

  minify() {
    const value = this.getValue()
    const minifiedValue = value.replace(/(\n|\s\s)/g, '')
    if (!minifiedValue) {
      return
    }
    this.setValue(minifiedValue)
    return minifiedValue
  }

  markEnter(value: string) {
    const newVal = value.replace(/\n/g, '\\n')
    this.setValue(newVal)
  }

  format() {
    const minifiedValue = this.getMinifiedValue()
    if (!minifiedValue) {
      return
    }
    const prettiered = prettier.format(minifiedValue, {
      parser: 'html',
      plugins: [prettierHtml],
      htmlWhitespaceSensitivity: 'ignore',
      printWidth: 9999,
    })
    this.setValue(prettiered)
  }

  dispose() {
    this.editor.dispose()
  }
}
```

这里将monoco-editor实例封装成一个对象
1. 初始化时调用monaco-editor内置worker方法
2. 设置数据操作的model，并绑定到后续初始化的editor中，这样model就能操作editor里面的数据了。分离editor ui配置以及model数据操作有效坚守逻辑与ui耦合。
3. 新增插入逻辑，将文本数据按字符分割成数组，定位到editor中的position index，将text插入其中并生成换行后的文本内容

# 针对editor编辑器相关的操作
## 格式化/压缩/复制/清空 

```vue
<template>
  <div id="editor-container">
    <ul class="flex flex-wrap operators">
      <li>
        <a-button @click="handleFormat">格式化</a-button>
      </li>
      <li>
        <a-button @click="handleMinify">压缩</a-button>
      </li>
      <li>
        <a-button @click="handleCopy">复制</a-button>
      </li>
      <li>
        <a-button @click="handleClear">清空</a-button>
      </li>
      <li>
        <div class="flex minify-mode">
          <span>精简模式</span>
          <a-switch v-model:checked="checked" @change="handleMiniMode" />
        </div>
      </li>
      <li>
        <a-button @click="handlePreview">预览</a-button>
      </li>
    </ul>
    <div ref="codeEditBox" class="editor-box"></div>
    <a-modal v-model:visible="isClearDialogShow" title="警告" @ok="handleClearOk">
      <p>确认要清空吗？</p>
    </a-modal>
  </div>
</template>
<script setup lang="ts">
  import { onBeforeUnmount, onMounted, ref } from 'vue'
  import Editor from './utils/editor'
  import { message } from 'ant-design-vue'
  import { ssmlEditorHandler } from './utils/editor-helper'
  const emit = defineEmits(['preview'])
  const changeValue = (value: string) => {
    console.log(value)
  }
  let editor
  const cb = navigator.clipboard
  const isClearDialogShow = ref(false)
  const checked = ref(false)
  const handleFormat = () => {
    editor.format()
  }
  const handleMinify = () => {
    editor.minify()
  }
  const handleCopy = () => {
    const data = editor.getValue()
    cb.writeText(data)
    message.info('已复制')
  }
  const handleClear = () => {
    isClearDialogShow.value = true
  }
  const handleClearOk = () => {
    editor.setValue('')
    isClearDialogShow.value = false
  }
  const handleMiniMode = (isSimplyfy: boolean) => {
    if (isSimplyfy) {
      editor.minify()
      ssmlEditorHandler.init(editor.getValue())
      editor.disable()
      const value = ssmlEditorHandler.simplify()
      editor.setValue(value)
    } else {
      editor.enable()
      editor.setValue(ssmlEditorHandler.restore())
    }
    editor.format()
  }
  const handlePreview = () => {
    emit('preview',editor.getValue())
  }
  onMounted(() => {
    editor = new Editor('.editor-box', changeValue)
  })
  onBeforeUnmount(() => {
    editor.dispose()
  })
</script>
<style lang="scss" scoped>
  #editor-container {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    .operators {
      justify-content: flex-start;
      align-items: center;
      gap: 10px;
    }
    .minify-mode {
      align-items: center;
      gap: 5px;
    }
  }
  .editor-box {
    flex: 1;
    width: 100%;
  }
</style>
```

## 精简化

src/components/monaco-editor/utils/editor-helper.ts
```typescript
// 处理粘贴的发音内容
export function getPasteNodes(pastedSting: string) {
  if (pastedSting === '') {
    return []
  }
  const fragment = document.createRange().createContextualFragment(pastedSting)
  const firstNode = fragment.childNodes[0] as Element
  // TODO: 没找到粘贴错误文本的规律，待优化
  if (firstNode.tagName !== 'DIV') {
    return [...Array.from(fragment.childNodes)]
  }
  console.error('请输入正确发音内容')
  return []
}
// 处于 <data></data> 中的标签不删除
function isRemoveNode(node: Element, parentNode: ParentNode): boolean {
  const nodeName = node.nodeName
  const parentNodeName = (parentNode as HTMLElement).nodeName
  if (nodeName === 'name' && parentNodeName === 'data') {
    return false
  }
  return true
}
class Xml {
  xml: Document
  constructor(ssml: string) {
    const domParser = new DOMParser()
    this.xml = domParser.parseFromString(ssml, 'text/xml')
  }
  removeTags(tagNames: string[]) {
    for (let i = 0; i < tagNames.length; i += 1) {
      const tagName = tagNames[i]
      const nodes = this.xml.getElementsByTagName(tagName)
      const length = nodes.length
      for (let j = length - 1; j > -1; j -= 1) {
        const node = nodes[j]
        const parentNode = node.parentNode
        if (parentNode && isRemoveNode(node, parentNode)) {
          // ? removeChild 会导致 nodes 变化
          parentNode.removeChild(node)
        }
      }
    }
  }
  getAttribute(payload: {
    attribute_name: string
    default_value: string
  }) {
    const node = this.xml.getElementsByTagName('speak')[0]
    if (node) {
      return node.getAttribute(payload.attribute_name)
    }
    return payload.default_value
  }
  innerHTML() {
    return (new XMLSerializer()).serializeToString(this.xml)
  }
}
// ssml 编辑模式下，精简、还原处理器
class SsmlEditorHandler {
  xml: Xml | null
  source: string
  init(content: string) {
    this.source = content
    this.xml = new Xml(content)
  }
  simplify() {
    if (this.xml) {
      this.xml.removeTags([
        'user-name',
        'user-type',
        'image-name',
        'name',
        'icon',
        'action-name',
        'action-image',
        'action-video',
        'action-from',
        'action-offset',
      ])
      return this.xml.innerHTML()
    }
    return this.source
  }
  restore() {
    return this.source
  }
  // 获取 speak 标签属性值
  attr() {
    if (!this.xml) {
      return {
        subtitle: 'autoUI',
        speed: '1',
        pitch: '1',
        volume: '1',
      }
    }
    const subtitle = this.xml.getAttribute({
      attribute_name: 'sub-title',
      default_value: 'autoUI',
    })
    const speed = this.xml.getAttribute({
      attribute_name: 'pitch',
      default_value: '1',
    })
    const pitch = this.xml.getAttribute({
      attribute_name: 'speed',
      default_value: '1',
    })
    const volume = this.xml.getAttribute({
      attribute_name: 'volume',
      default_value: '1',
    })
    return {
      subtitle,
      speed,
      pitch,
      volume,
    }
  }
}
export const ssmlEditorHandler = new SsmlEditorHandler()
```

这里增加了一个增加了一个语料（ssml）编辑处理对象，用于处理xml语言中文本内容相关格式化/精简化的内容

```vue
<script>

// 新增精简化逻辑
const handleMiniMode = (isSimplyfy:boolean) => {
	if(isSimplyfy) {
	  editor.minify()
	  ssmlEditorHandler.init(editor.getValue())
	  editor.disable()
	  const value = ssmlEditorHandler.simplify()
	  editor.setValue(value)
	}
	else {
	  editor.enable()
	  editor.setValue(ssmlEditorHandler.restore())
	}
	editor.format()
}

</script>

```

