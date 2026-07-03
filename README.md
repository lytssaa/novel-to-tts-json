# novel-to-tts-json

将 TXT 小说逐章转换为 [Fish Audio S2.1 Pro](https://fish.audio) 有声书 JSON 脚本的 WorkBuddy Skill。

逐章转换 + **fidelity gate** 末句比对 → 零内容丢失。断点续传，不怕中断。

## 功能

| 功能 | 说明 |
|------|------|
| 📖 章节切分 | 自动识别「第X章/Chapter X」，切成独立文件 |
| 🔄 逐章转换 | AI 按 Fish Audio 规范逐章输出结构化 JSON |
| 🛡️ fidelity gate | 每章 TXT 末句 vs JSON 末句关键词比对，硬失败拦截 + 警告分级 |
| 📊 断点续传 | `progress.json` 记录进度，中断后接着来 |
| 📋 批量检查 | 一次性检查全部章节转换质量 |

## 快速开始

### 前置

- WorkBuddy（已安装 `novel-to-tts-json` skill）
- Python 3.8+（仅标准库，零依赖）

### 1. 章节切分（如果 TXT 是整本的话）

```bash
python scripts/chapter_tools.py split 联镖记.txt ./各章拆分/
```

### 2. 在 WorkBuddy 中触发转换

直接说一句话：

```
用 novel-to-tts-json 转换 C:/Users/17165/Desktop/联镖记/各章拆分/
```

或更短的触发词也能识别：

```
转有声书
继续转
接着做
txt 转 json
```

Skill 会自动：
- 扫描章节，读取 `references/fish-audio-prompt.md` 规范
- 逐章让 AI 转换 JSON（角色映射 + 台词脚本）
- 每章跑 fidelity gate 验证
- 保存进度到 `progress.json`

### 3. 手动检查

```bash
# 单章检查
python scripts/chapter_tools.py check 第一章上.txt output/联镖记_001_第一章上.json

# 批量检查
python scripts/chapter_tools.py batch_check ./各章拆分/ ./各章拆分/output/

# 查看转换进度
python scripts/chapter_tools.py list ./各章拆分/ ./各章拆分/output/

# 进度管理
python scripts/chapter_tools.py progress load ./output/
python scripts/chapter_tools.py progress save ./output/ 3 "第三章 夜探敌营"
```

### fidelity gate 输出示例

```
============================================================
FIDELITY GATE 检查结果
============================================================
状态: ✓ 通过
TXT 末句:  ...小白龙冷笑一声，翻身上马，消失在夜色中。
JSON 末句: ...小白龙冷笑一声，翻身上马，消失在夜色中。
TXT 台词数:  35
JSON 台词数: 56 (偏差 21)
末句重叠率: 100%

警告（不阻断）:
  - 台词数量有偏差(偏差21)，但末句匹配，可能因角色长篇叙述嵌套引号导致
============================================================
```

## JSON 输出格式

```json
{
  "character_map": {
    "林廷扬": {"gender": "男", "age": "中年", "tone": "沉稳"},
    "小白龙": {"gender": "男", "age": "青年", "tone": "狂傲"}
  },
  "script": [
    {
      "speaker": "旁白",
      "content": "夜深人静，官道上马蹄声急。",
      "emo_vector": {"happy": 0.0, "sad": 0.0, "angry": 0.0, "fear": 0.0, "surprise": 0.0},
      "delay": 800
    },
    {
      "speaker": "林廷扬",
      "content": "来者何人？",
      "emo_vector": {"happy": 0.0, "sad": 0.0, "angry": 0.1, "fear": 0.0, "surprise": 0.1},
      "delay": 500
    }
  ]
}
```

字段约束：
- `speaker`：引号内容 → 角色名，无引号 → `"旁白"`
- `content`：≤120 汉字，超长在标点处断开
- `delay`：旁白 800 / 对话 500 / 转折 1500 / 场景切换 2000
- `emo_vector`：5 维向量，每维 0.0~1.0

## 文件结构

```
novel-to-tts-json/
├── SKILL.md                          # 主 skill 文件
├── references/
│   └── fish-audio-prompt.md          # 完整转换规范（AI 逐章参考）
├── scripts/
│   └── chapter_tools.py              # 切分 + fidelity gate + 进度工具
└── README.md
```

## License

MIT
