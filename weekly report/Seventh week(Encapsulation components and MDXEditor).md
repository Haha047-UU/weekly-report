# Encapsulation components and MDXEditor
## Encapsulation components
生成功能函数组件，允许你为组件指定props，可以获得更好的类型检查和编辑器支持。
```
const GithubExplorer: React.FC = () => {
  
};
export default GithubExplorer;

```

```
const [username, setUsername] = useState('');
  const [repos, setRepos] = useState<Repo[]>([]);
  const [selectedRepo, setSelectedRepo] = useState<string | null>(null);
  const [selectedFileContent, setSelectedFileContent] = useState<string | null>(null);
  const [defaultBranch, setDefaultBranch] = useState<string | null>(null);
  const [treeData, setTreeData] = useState<any>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  //采用了React的useState钩子，前面创建的对象是状态变量，后值的函数用于更新状态变量的值，结果可以指定类型被存储。
```
之后封装之前创建的api，成为组件。

## MDXEditor
### 初始化MDXEditor
导入相应的插件
```
import {
  headingsPlugin,
  listsPlugin,
  quotePlugin,
  thematicBreakPlugin,
  markdownShortcutPlugin,
  codeBlockPlugin,
  codeMirrorPlugin,
  sandpackPlugin,
  linkPlugin,
  MDXEditor,
  toolbarPlugin,
  KitchenSinkToolbar,
  SandpackConfig,
  type MDXEditorMethods,
  type MDXEditorProps,
} from '@mdxeditor/editor'
```
导出一个组件,返回JSX元素，用于渲染MDXEditor组件
```
export default function InitializedMDXEditor({
  editorRef,
  ...props
}: { editorRef: ForwardedRef<MDXEditorMethods> | null } & MDXEditorProps) {
  return (
    <>
      <MDXEditor
        onChange={console.log}
        plugins={[
          toolbarPlugin({ toolbarContents: () => <KitchenSinkToolbar /> }),
          headingsPlugin(),
          listsPlugin(),
          quotePlugin(),
          thematicBreakPlugin(),
          markdownShortcutPlugin(),
          codeBlockPlugin({ defaultCodeBlockLanguage: '' }),
          sandpackPlugin({ sandpackConfig: simpleSandpackConfig }),
          codeMirrorPlugin({ codeBlockLanguages: { js: 'JavaScript', css: 'CSS', txt: 'Plain Text', tsx: 'TypeScript', '': 'Unspecified' } }),
          linkPlugin(),
        ]}
        {...props}
        ref={editorRef}
      />
    </>
  )
}
```

### 转发MDXEditor
```
'use client'

import dynamic from 'next/dynamic'
import { forwardRef } from 'react'
import {
  type MDXEditorMethods,
  type MDXEditorProps
} from '@mdxeditor/editor'
import '@mdxeditor/editor/style.css'

const Editor = dynamic(() => import('./InitializedMDXEditor'), {//ssr设置为false，表示的是在服务器端渲染时不加载组件。
  ssr: false
})

// This is what is imported by other components. Pre-initialized with plugins, and ready
// to accept other props, including a ref.
export const ForwardRefEditor = forwardRef<MDXEditorMethods, MDXEditorProps>((props, ref) => <Editor {...props} editorRef={ref} />)

// TS complains without the following line
ForwardRefEditor.displayName = 'ForwardRefEditor'
//将 props 和 ref 传递给动态导入的 Editor 组件。引用转发：editorRef={ref}：将传入的 ref 传递给 Editor 组件的 editorRef 属性。
```

### 使用MDXEditor
```
import { ForwardRefEditor } from './ForwardRefEditor';

{(selectedFileContent &&
<ForwardRefEditor markdown={selectedFileContent} />         
          )}
```