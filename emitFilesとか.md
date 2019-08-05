transformerを取得してemitFilesへ
const transformers: TransformerFactory<SourceFile | Bundle>[] = [];

export function getTransformers(compilerOptions: CompilerOptions, customTransformers?: CustomTransformers, emitOnlyDtsFiles?: boolean): EmitTransformers {
    return {
        scriptTransformers: getScriptTransformers(compilerOptions, customTransformers, emitOnlyDtsFiles),
        declarationTransformers: getDeclarationTransformers(customTransformers),
    };
}

function transformNodes<T extends Node>(resolver: EmitResolver | undefined, host: EmitHost | undefined, options: CompilerOptions, nodes: ReadonlyArray<T>, transformers: ReadonlyArray<TransformerFactory<T>>, allowDtsFiles: boolean): TransformationResult<T> {
/* 中略 */
  // Chain together and initialize each transformer.
  const transformersWithContext = transformers.map(t => t(context));
  const transformation = (node: T): T => {
      for (const transform of transformersWithContext) {
          node = transform(node);
      }
      return node;
  };

  // prevent modification of transformation hooks.
  state = TransformationState.Initialized;

  // Transform each node.
  const transformed = map(nodes, allowDtsFiles ? transformation : transformRoot);
/* 中略 */
}

transformNodes(..., scriptTransformers)
emitJsFileOrBundle
emitSourceFileOrBundle
forEachEmittedFile
emitFiles