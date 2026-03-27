# Travel Planner Skill

<div align="center">

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Manus Skill](https://img.shields.io/badge/Manus-Skill-purple.svg)](https://manus.im)

一个用于自动生成个人定制旅行计划 Excel 表格的 Manus Skill

[English](#english) | [中文](#中文)

</div>

---

## 中文

### 🌟 功能特性

- 🗺️ **智能行程规划** - 根据目的地、天数和个人偏好自动生成每日行程
- 🏨 **住宿推荐** - 提供中高端酒店推荐及实时价格参考
- ✈️ **机票参考** - 自动搜索并整理去程、回程及中转航班信息
- 🎒 **动态物品清单** - 根据目的地、季节和行程特点智能生成行李清单
- 🏛️ **深度景点介绍** - 提供详细的景点历史背景、打卡路线和实用信息
- 📊 **精美 Excel 输出** - 自动生成包含 5 个 Sheet 的专业旅行计划表，排版精美

### 🚀 快速开始

#### 安装

**作为 Manus Skill 安装：**

```bash
# 克隆仓库到 Manus skills 目录
cd ~/skills
git clone https://github.com/kkkaaayla2/travel-planner-skill.git travel-planner
```

#### 使用方法

在与 Manus 交互时，只需提及以下内容即可触发此 Skill：
- "我想去[目的地]旅行"
- "帮我规划一下去[目的地]的行程"
- "制作一个[目的地][天数]天的旅行计划表"
- 或直接上传旅行笔记、Excel、PDF 或网页链接，Manus 会自动解析并生成计划。

### 📁 项目结构

```
travel-planner-skill/
├── SKILL.md                    # Manus Skill 主文档，包含所有提示词和逻辑
├── README.md                   # 项目说明（本文件）
└── LICENSE                     # MIT 许可证
```

### ⚠️ 注意事项

- 本 Skill 依赖于部分免费 API（如 Frankfurter 汇率、Open-Meteo 天气）和网页搜索功能。
- 生成的 Excel 文件将保存在 `/mnt/user-data/outputs/` 目录下。

### 🤝 贡献

欢迎贡献！请随时提交 Pull Request 或创建 Issue。

### 📄 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件

---

## English

### 🌟 Features

- 🗺️ **Smart Itinerary Planning** - Automatically generates daily itineraries based on destination, duration, and personal preferences.
- 🏨 **Accommodation Recommendations** - Provides mid-to-high-end hotel recommendations with real-time price references.
- ✈️ **Flight References** - Automatically searches and organizes outbound, return, and transit flight information.
- 🎒 **Dynamic Packing List** - Intelligently generates a packing list based on destination, season, and itinerary characteristics.
- 🏛️ **In-depth Attraction Guides** - Offers detailed historical backgrounds, photo routes, and practical information for attractions.
- 📊 **Beautiful Excel Output** - Automatically generates a professional travel plan spreadsheet with 5 beautifully formatted sheets.

### 🚀 Quick Start

#### Installation

**Install as Manus Skill:**

```bash
# Clone to Manus skills directory
cd ~/skills
git clone https://github.com/kkkaaayla2/travel-planner-skill.git travel-planner
```

#### Usage

Simply mention the following when interacting with Manus to trigger this Skill:
- "I want to travel to [Destination]"
- "Help me plan a trip to [Destination]"
- "Create a [Duration]-day travel plan for [Destination]"
- Or directly upload travel notes, Excel, PDF, or web links, and Manus will automatically parse and generate the plan.

### 📁 Project Structure

```
travel-planner-skill/
├── SKILL.md                    # Main Manus Skill document containing all prompts and logic
├── README.md                   # Project documentation (this file)
└── LICENSE                     # MIT License
```

### ⚠️ Disclaimer

- This Skill relies on some free APIs (e.g., Frankfurter for exchange rates, Open-Meteo for weather) and web search functionalities.
- The generated Excel file will be saved in the `/mnt/user-data/outputs/` directory.

### 🤝 Contributing

Contributions are welcome! Feel free to submit Pull Requests or create Issues.

### 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

---

<div align="center">

**⭐ If you find this project helpful, please give it a star! ⭐**

Made with ❤️ for the Manus community

</div>
