# 一个完整的SKILL
每个 skill 都是一个以 SKILL.md 作为入口点的目录(**较为完整的SKILL格式**)：
```txt
my-skill/
├── SKILL.md           # 主要说明（必需） (required - overview and navigation)
├── template.md        # Claude 要填写的模板
├── reference.md       # (detailed API docs - loaded when needed)
├── examples/          # (usage examples - loaded when needed)
│   └── sample.md      # 显示预期格式的示例输出
└── scripts/
    └── validate.sh    # Claude 可以执行的脚本
    └── helper.py      # (utility script - executed, not loaded)
```

从 SKILL.md 中引用支持文件，以便 Claude 知道每个文件包含什么以及何时加载它：
```txt
## Additional resources

- For complete API details, see [reference.md](reference.md)
- For usage examples, see [examples.md](examples.md)
```

- template.md 就像是给 Claude 发了一张“标准试卷”，它只需要负责把答案（具体的业务逻辑）填进对应的格子里。这在编写 PRD（产品需求文档）、Post-mortem（事故复盘报告）、或是标准化的 Commit Message 时特别好用！
  + template.md 示例: 参考:[template.md](./000.template.md.md)

SKILL.md 包含主要说明，是必需的。其他文件是可选的，让你构建更强大的 skills：Claude 要填写的模板、显示预期格式的示例输出、Claude 可以执行的脚本或详细的参考文档。从你的 SKILL.md 中引用这些文件，以便 Claude 知道它们包含什么以及何时加载它们。