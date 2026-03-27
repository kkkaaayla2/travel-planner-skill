# Travel Planner Skill

<div align="center">

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Manus Skill](https://img.shields.io/badge/Manus-Skill-purple.svg)](https://manus.im)

一个用于自动生成个人定制旅行计划 Excel 表格的 Manus Skill

[English](#english) | [中文](#中文)

</div>

---

## 中文

### 🌟 为什么你需要这个 Skill？

**爱旅游的J人有福了！**

**和我一样有刁钻的旅游偏好的人更有福了！**

J人一定能懂每个行程都要精密计算安排时间、反复在Google map、xhs、预定网站反复跳转确认的痛苦。

这个skill会先询问用户行程偏好，让我选择了希望去的城市和景点。然后完美生成了有5个sheet（行程、住宿推荐、机票参考、物品清单、景点介绍）的Excel表！

所有的行程都包括详细的、具有时效性的导航路线，例如：

> **基督城大教堂广场（Cathedral Square）**
> 打卡路线：广场中心 -> 基督城大教堂（正在修复中，外观拍照）-> 过渡纸板教堂（Cardboard Cathedral）内部参观

**J人狂喜！**

也有朋友问我，为什么不直接用携程/圆周旅迹AI规划呢？我说你一定是个P人。虽然他们在产品层面上，有更好的交互体验和图片介绍，但是只给出景点和建议游玩时间，而不给出具体的时间段；完全不考虑我的个人偏好，一天6个行程，我看一眼就晕过去了；生成的一长串富文本其实还不如Excel结构化和细节丰富，而且国外容易信号差，我必须有本地文件才安心。

### 🚀 快速开始

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

### 🌟 Why You Need This Skill

**Good news for travel-loving 'J' personalities (Judging in MBTI)!**

**Even better news for those who have picky travel preferences like me!**

'J' personalities will definitely understand the pain of having to precisely calculate and schedule time for every itinerary, and repeatedly jumping between Google Maps, social media, and booking websites to confirm details.

This skill will first ask about the user's travel preferences, letting me choose the cities and attractions I want to visit. Then it perfectly generates an Excel spreadsheet with 5 sheets (Itinerary, Accommodation Recommendations, Flight References, Packing List, Attraction Guides)!

All itineraries include detailed, time-sensitive navigation routes, for example:

> **Christchurch Cathedral Square**
> Photo Route: Square Center -> Christchurch Cathedral (under restoration, exterior photo) -> Cardboard Cathedral (interior visit)

**'J' personalities rejoice!**

Some friends also asked me, why not just use other AI travel planners? I said you must be a 'P' personality (Perceiving). Although they might have better interactive experiences and picture introductions at the product level, they only provide attractions and suggested play times, without giving specific time slots; they completely ignore my personal preferences, 6 itineraries a day, I pass out just looking at it; the long string of rich text generated is actually not as structured and detail-rich as Excel, and the signal is often poor abroad, I must have a local file to feel at ease.

### 🚀 Quick Start

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
