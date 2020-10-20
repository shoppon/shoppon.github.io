# Clang-format使用

## 安装

clang-format是```LLVM```编译器自带的功能，使用clang-format需安装```LLVM```。

## 配置

clang-format工具支持根据```.clang-format```文件来格式化代码，根据《华为C++语言通用编程规范_V3.3》抽象出的```.clang-format```样例配置如下：

```shell
---
Language:        Cpp
BasedOnStyle:  Google
AccessModifierOffset: -4
AlignAfterOpenBracket: DontAlign
AlignConsecutiveAssignments: false
AlignConsecutiveDeclarations: false
AlignEscapedNewlines: Right
AlignOperands:   true
AlignTrailingComments: true
AllowAllParametersOfDeclarationOnNextLine: true
AllowShortBlocksOnASingleLine: true
AllowShortCaseLabelsOnASingleLine: false
AllowShortFunctionsOnASingleLine: None
AllowShortIfStatementsOnASingleLine: false
AllowShortLoopsOnASingleLine: false
AlwaysBreakAfterDefinitionReturnType: None
AlwaysBreakAfterReturnType: None
AlwaysBreakBeforeMultilineStrings: false
AlwaysBreakTemplateDeclarations: true
BinPackArguments: false
BinPackParameters: true
BraceWrapping:
  AfterClass:      false
  AfterControlStatement: false
  AfterEnum:       false
  AfterFunction:   true
  AfterNamespace:  false
  AfterObjCDeclaration: false
  AfterStruct:     false
  AfterUnion:      false
  AfterExternBlock: false
  BeforeCatch:     false
  BeforeElse:      false
  IndentBraces:    false
  SplitEmptyFunction: false
  SplitEmptyRecord: false
  SplitEmptyNamespace: false
BreakBeforeBinaryOperators: None
BreakBeforeBraces: Custom
BreakBeforeInheritanceComma: false
#BreakInheritanceList: AfterColon
BreakBeforeTernaryOperators: true
BreakConstructorInitializersBeforeComma: false
BreakConstructorInitializers: BeforeColon
BreakAfterJavaFieldAnnotations: true
BreakStringLiterals: false
ColumnLimit:     120
CommentPragmas:  '^lint'
CompactNamespaces: false
ConstructorInitializerAllOnOneLineOrOnePerLine: true
ConstructorInitializerIndentWidth: 4
ContinuationIndentWidth: 4
Cpp11BracedListStyle: true
DerivePointerAlignment: false
DisableFormat:   false
ExperimentalAutoDetectBinPacking: false
FixNamespaceComments: true
ForEachMacros:
  - foreach
  - Q_FOREACH
  - BOOST_FOREACH
IncludeBlocks:   Regroup
IncludeCategories:
  - Regex:           '^<ext/.*\.h>'
    Priority:        2
  - Regex:           '^<.*\.h>'
    Priority:        1
  - Regex:           '^<.*'
    Priority:        2
  - Regex:           '.*'
    Priority:        3
IncludeIsMainRegex: '(Test)?$'
IndentCaseLabels: true
IndentPPDirectives: None
IndentWidth:     4
IndentWrappedFunctionNames: true
JavaScriptQuotes: Leave
JavaScriptWrapImports: true
KeepEmptyLinesAtTheStartOfBlocks: true
MacroBlockBegin: ''
MacroBlockEnd:   ''
MaxEmptyLinesToKeep: 1
NamespaceIndentation: None
ObjCBlockIndentWidth: 4
ObjCSpaceAfterProperty: false
ObjCSpaceBeforeProtocolList: false
PenaltyBreakAssignment: 100
PenaltyBreakBeforeFirstCallParameter: 1
PenaltyBreakComment: 200
PenaltyBreakFirstLessLess: 120
PenaltyBreakString: 10
#PenaltyBreakTemplateDeclaration: 0
PenaltyExcessCharacter: 1000000
PenaltyReturnTypeOnItsOwnLine: 200
PointerAlignment: Left
ReflowComments:  true
SortIncludes:    true
SortUsingDeclarations: true
SpaceAfterCStyleCast: false
SpaceAfterTemplateKeyword: false
SpaceBeforeAssignmentOperators: true
#SpaceBeforeCpp11BracedList: true
#SpaceBeforeCtorInitializerColon: true
#SpaceBeforeInheritanceColon: true
#SpaceBeforeParens: ControlStatements
#SpaceBeforeRangeBasedForLoopColon: true
SpaceInEmptyParentheses: false
SpacesBeforeTrailingComments: 2
SpacesInAngles: false
SpacesInContainerLiterals: false
SpacesInCStyleCastParentheses: false
SpacesInParentheses: false
SpacesInSquareBrackets: false
Standard:        Cpp11
TabWidth:        4
UseTab:          Never
...
```

## 批量格式化

可在linux环境上通过clang-format批量格式化工程代码，具体步骤如下：

1. 在工程根目录创建```.clang-format```文件，内容如下章节。
2. 执行```find . -name *.h -o -name *.cpp|xargs clang-format -style=file -i```一键格式化所有代码。
