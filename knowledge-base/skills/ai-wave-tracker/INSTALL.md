# 🌊 AI浪潮产业链追踪 Skill — 安装指南

## 快速安装

```bash
# 方式一：从 GitHub 克隆后复制
git clone https://github.com/Saminar/knowledge-base.git /tmp/kb-temp
cp -r /tmp/kb-temp/skills/ai-wave-tracker ~/.workbuddy/skills/
rm -rf /tmp/kb-temp

# 方式二：直接下载（如果已 clone 本仓库）
cp -r skills/ai-wave-tracker ~/.workbuddy/skills/
```

## 安装依赖

```bash
pip3 install requests akshare
```

## 配置（可选）

修改 `~/.workbuddy/skills/ai-wave-tracker/scripts/sync_report.sh` 中的：
- `KB_DIR`：你的 knowledge-base 本地路径

## 触发方式

在对话中输入以下任意关键词，skill 将自动激活：

- `AI浪潮分析`
- `AI产业链追踪`
- `港股AI分析`
- `AI芯片行情`
- `光模块分析`

## 手动运行

```bash
SKILL_DIR=~/.workbuddy/skills/ai-wave-tracker/scripts
DATE=$(date +%Y-%m-%d)

# Step 1: 获取行情数据
python3 $SKILL_DIR/fetch_stock_data.py --output /tmp/aiwave_stock.json

# Step 2: 获取新闻与主题
python3 $SKILL_DIR/fetch_news_analysis.py --output /tmp/aiwave_news.json

# Step 3: 生成 HTML 报告
bash $SKILL_DIR/generate_report.sh /tmp/aiwave_stock.json /tmp/aiwave_news.json $DATE

# Step 4: 同步到 GitHub
bash $SKILL_DIR/sync_report.sh /tmp/aiwave_report_${DATE}.html $DATE
```

## 产出物

- 60KB 暗色主题 HTML 报告（7 Tab + 11 ECharts 图表）
- 覆盖港股/A股/美股 35+ 核心 AI 产业链标的
- 自动推送至 GitHub knowledge-base `docs/investment/`

## 分析框架

### 四象限产业透视法
| 象限 | 领域 | 代表标的 |
|------|------|---------|
| 一 | 算力底座（GPU/代工/HBM） | NVDA, 中芯, 寒武纪 |
| 二 | 数据通路（光模块/网络） | 中际旭创, 天孚通信 |
| 三 | 算法平台（云/大模型） | 阿里, 腾讯, 百度 |
| 四 | 应用落地（AI Server/PC/Phone） | 联想, 浪潮, 工业富联 |

### 五维评估
技术壁垒(40%) × 市场地位 × AI受益度 × 政策顺风 × 估值安全边际
