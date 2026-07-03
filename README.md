# novel-to-tts-json

将 TXT 小说逐章转换为 [Fish Audio S2.1 Pro](https://fish.audio) 有声书 JSON 脚本的 WorkBuddy Skill。

**AI 推理驱动**，逐章转换 + **fidelity gate** 末句比对 → 零内容丢失。断点续传，不怕中断。

## 功能

| 功能 | 说明 |
|------|------|
| 📖 章节切分 | 自动识别「第X章/Chapter X」，切成独立文件 |
| 🧹 非故事内容过滤 | 自动跳过喜马拉雅播客头尾、广告推广语 |
| 🔄 AI 推理转换 | AI 逐句阅读理解文本，自行识别角色、语气、切分、delay |
| 🛡️ fidelity gate | 每章故事末句 vs JSON 末句关键词比对，硬失败拦截 + 警告分级 |
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

或更短的触发词：

```
转有声书 / 继续转 / 接着做 / txt 转 json
```

Skill 会自动：
- 扫描章节，加载转换规范
- **逐句阅读全文，AI 推理**识别角色、判断语气、拆分句式
- 输出含 character_map + script 的纯 JSON
- 每章跑 fidelity gate 验证
- 保存进度

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

## JSON 输出格式

```json
{
  "character_map": {
    "林廷扬": {
      "gender": "男",
      "age": "中年",
      "role_tag": "重要角色",
      "personality": "安远镖局总镖头，武艺高强，为人正直沉稳。",
      "timbre": "低沉有力，中年男性的沉稳嗓音，略带威严。"
    }
  },
  "script": [
    {
      "speaker": "旁白",
      "content": "夜深人静，官道上马蹄声急。",
      "emo_vector": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0],
      "delay": 800
    },
    {
      "speaker": "林廷扬",
      "content": "来者何人？",
      "emo_vector": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0],
      "delay": 500
    }
  ]
}
```

字段约束：
- `character_map`：5 个字段（gender/age/role_tag/personality/timbre）
- `script` 每条：speaker / content / emo_vector / delay
- `speaker`：引号内容 → 角色名，无引号 → `"旁白"`
- `content`：≤120 汉字，超长在标点处断开
- `delay`：旁白 800 / 对话 500 / 转折 1500 / 场景切换 2000
- `emo_vector`：**固定 8 元素数组** `[0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0]`

## 核心设计

### AI 推理 vs 脚本生成

本 skill 的核心是 **AI 推理**，不是模板拼接：

- ✅ AI 逐句阅读理解文本，自行判断角色归属、语气、切分
- ✅ 每个 TXT 独立处理，不需要参照已有 JSON
- ❌ 禁止编写 Python 脚本批量生成 JSON
- ❌ 禁止参照已有 JSON "对齐格式"

### 非故事内容过滤

TXT 文件常混入播客平台头尾语（喜马拉雅格式），skill 会在转换前自动识别并跳过：
- `欢迎您收听由喜马拉雅出品…`
- `听众朋友，本集已播讲完毕…`
- `请订阅专辑，下集精彩继续`

## 文件结构

```
novel-to-tts-json/
├── SKILL.md                          # 主 skill 文件
├── references/
│   └── fish-audio-prompt.md          # 完整转换规范 + 常见错误自查表
├── scripts/
│   └── chapter_tools.py              # 切分 + 非故事过滤 + fidelity gate + 进度工具
└── README.md
```

## License

MIT
