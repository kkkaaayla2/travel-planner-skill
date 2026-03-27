---
name: travel-planner
description: >
  生成个人定制旅行计划Excel表格（多Sheet）。当用户提到想去某个地方旅行、规划行程、
  制作旅行计划表，或输入目的地+天数时，立刻使用此skill。
  即使用户说"帮我排一下行程"、"去XX玩几天"、"做个旅行表格"等口语表达也应触发。
  如果用户上传了旅行笔记、Excel、PDF或给出网页链接，也应使用此skill解析后再生成Excel。
  输出包含5个Sheet：行程 / 住宿推荐 / 机票参考 / 物品清单 / 景点介绍。
---

# Travel Planner Skill

---

## 用户旅行偏好画像（固定规则，每次都要遵守）

- **景点偏好：经典地标 + 拍照优先**
  喜欢iconic、视觉冲击力强、适合拍照的地标（如帝国大厦、自由女神像）。
  不喜欢过于小众的街区漫游或精品市集。每个城市优先安排最有代表性的"必打卡"景点。

- **餐厅：多元化，不全是西餐**
  每次行程中至少安排 1 餐中餐、1 餐日料或亚洲菜（泰、越、韩等）。
  brunch 咖啡馆可以是西式，但正餐要多样化。

- **购物限制**：最多安排 1 个半天用于逛街/购物，除非目的地是日本、韩国等购物型城市。

- **主要语言为中文**：表格所有内容（表头、行程、备注、提示区、景点介绍等）均使用中文。景点英文名可附在中文后作为导航参考，但主体文字必须是中文。

- **禁用 emoji**：表格所有单元格禁止使用任何 emoji 表情符，包括表头、备注、提示区。

- **字体规范**：
  - 中文内容：宋体（`Font(name="宋体")`）
  - 英文内容（地址、网址、航班号等）：Times New Roman（`Font(name="Times New Roman")`）
  - 中英文混排单元格：统一使用宋体（宋体对英文有合理 fallback 渲染）

- **行高自适应**：所有数据行不使用固定行高，必须根据单元格内容长度和列宽动态估算高度，确保内容完整显示。估算方法：
  ```python
  def est_height(text, col_width, base_h=16, min_h=20):
      if not text: return min_h
      lines = str(text).count('\n') + 1
      chars_per_line = max(1, int(col_width * 1.7))
      wrapped = sum((len(line) // chars_per_line + 1)
                    for line in str(text).split('\n'))
      return max(min_h, wrapped * base_h)
  ```
  每行取该行所有单元格估算高度的最大值。

- **F列行程规划格式**：
  - 景点必须给出具体打卡路线，格式：`[景点名称]\n打卡路线：[起点] -> [打卡点1] -> [打卡点2] -> [终点]`
  - 餐厅/咖啡馆前注明类型：`午餐：[名称]` / `咖啡：[名称]` / `晚餐：[名称]`

- 起床时间 10-11:00，出门约 11-12:00；晚上可排到 22-23:00；步行上限 20min
- 酒店偏好：中高端（4-5星 / 精品设计）；节奏宽松，每天 3-5 个活动
- 不显示预估费用

---

## Phase 0：正式生成前的必问问题

**在开始任何调研或生成之前，必须先完成以下所有确认。**

### 0A. 有无参考资料？

如果用户上传了 Excel / PDF / Word / 网页 URL，**优先处理这些参考资料**：
- `.xlsx/.xls`：用 openpyxl 或 pandas 读取
- `.pdf`：用 pdftotext 或 pypdf 提取文本
- `.docx`：用 pandoc 转 markdown
- 网页 URL：用 `web_fetch` 抓取正文
- 处理完后，向用户简述读到的内容，并确认哪些信息保留/修改

### 0B. 出发城市 & 返回城市

明确问清楚出发城市和返回城市（影响机票搜索、时差计算、物品清单中电器/签证需求）。

### 0C. 目的地（精确到城市）

不接受国家/大区，必须明确到城市。若不明确，主动给出城市推荐引导。
确认各城市分别待几天。

### 0D. 出发日期 & 天数

精确到年月日。若模糊，引导用户确认具体日期。

### 0E. 旅行偏好（结合目的地特点给选项）

根据目的地提供 2-4 个定制化选项引导用户选择，例如：

**示例：波士顿**
> "波士顿你更想侧重：
> A. 历史地标（Freedom Trail、波士顿茶党遗址）
> B. 学术校园（Harvard、MIT）
> C. 自然休闲（查尔斯河、公共花园）
> D. 美食体验（North End意大利街区、海鲜餐厅）"

**示例：纽约**
> "纽约你更想侧重：
> A. 经典地标拍照（Empire State、Brooklyn Bridge、Times Square）
> B. 顶楼观景（Summit One Vanderbilt、Top of Rock）
> C. 文化体验（百老汇演出、Met博物馆）
> D. 美食探索（各类亚洲料理、网红餐厅）"

---

## Phase 1：景点调研（生成行程前必须执行）

### 1A. 城市交通方式预研

搜索：`{城市} best way to get around for tourists 2025 2026`

**交通策略规则：**
- 纽约（Manhattan）：Uber/Lyft 优先，避免推荐地铁（换乘复杂，夜间安全差）
- 洛杉矶：纯 Uber/Lyft，绝不推荐公交
- 旧金山：Uber/Lyft 为主
- 波士顿：T 线 + Uber/Lyft
- 巴黎/伦敦/东京：地铁为主
- 国家公园/自然景区：提示租车/自驾

### 1B. 景点开放时间 & 推荐游览时长

**对每一个拟加入行程的景点，必须搜索：**
```
web_search: "{景点名} opening hours {year} visit duration"
```

重点核查：
- 是否有闭馆日（如 Summit One Vanderbilt 周二闭馆，部分博物馆周一闭馆）
- 开放时间段（如某些景点 16:00 提前关门）
- 推荐游览时长（合理分配时间，避免安排过紧或跑空）
- 是否需要提前预约/购票

**常见景点参考时长（搜索验证后使用）：**
- 自由女神像 + Ellis Island：全程 3-4 小时（含渡轮等待）
- Summit One Vanderbilt：2-3 小时（建议黄昏前入场）；周二闭馆
- 帝国大厦：1-2 小时；每日开放，建议避开 11am-3pm 高峰
- 百老汇演出（如狮子王）：约 2.5 小时（含中场休息）；晚场 7pm 约 9:30pm 结束
- Brooklyn Bridge 步行：单程约 30-40 分钟
- 时代广场：随时可去，夜晚最美，1 小时足够
- 波士顿 Freedom Trail：2.5-3 小时步行
- Harvard Yard 参观：1.5-2 小时

**根据搜索结果调整每天行程密度，禁止安排时间冲突或已知闭馆的景点。**

---

## Phase 2：Function Calls

### 2A. 实时汇率
```python
import urllib.request, json

def get_exchange_rate(from_currency, to_currency):
    """使用 Frankfurter API（免费，无需 Key）"""
    url = f"https://api.frankfurter.app/latest?from={from_currency}&to={to_currency}"
    with urllib.request.urlopen(url, timeout=10) as r:
        return json.loads(r.read())["rates"][to_currency]
# fallback: web_search "{from} {to} exchange rate today"
```

### 2B. 天气数据
```python
import urllib.request, json

def get_weather(lat, lon, dates):
    """Open-Meteo 免费 API，无需 Key"""
    url = (f"https://api.open-meteo.com/v1/forecast?"
           f"latitude={lat}&longitude={lon}"
           f"&daily=temperature_2m_max,temperature_2m_min,weathercode"
           f"&timezone=auto&start_date={min(dates)}&end_date={max(dates)}")
    with urllib.request.urlopen(url, timeout=10) as r:
        d = json.loads(r.read())["daily"]
    WMO = {range(0,1):"晴",range(1,4):"少云",range(4,10):"多云",
           range(45,68):"雾雨",range(61,68):"小雨",range(71,78):"雪",
           range(80,83):"阵雨",range(95,100):"雷雨"}
    def dec(c):
        for rng,lbl in WMO.items():
            if c in rng: return lbl
        return "多云"
    # 天气格式：用 C 代替度数符号，不使用任何 emoji
    return {d: f"{round(mx)}C / {round(mn)}C  {dec(wc)}"
            for d,mx,mn,wc in zip(d["time"],d["temperature_2m_max"],
                                   d["temperature_2m_min"],d["weathercode"])}
# 若超出16天预报范围，改用 web_search 搜历史气候均值手动填入
```

### 2C. 交通路线（Google Maps Directions API）
```python
import urllib.request, json, urllib.parse

def get_route(origin, destination, api_key, mode="transit"):
    """根据城市交通策略选择 mode：transit / driving / walking"""
    params = {"origin": origin, "destination": destination,
              "mode": mode, "language": "zh-CN", "key": api_key}
    url = "https://maps.googleapis.com/maps/api/directions/json?" + urllib.parse.urlencode(params)
    with urllib.request.urlopen(url, timeout=10) as r:
        data = json.loads(r.read())
    if data["status"] != "OK": return {"error": data["status"]}
    leg = data["routes"][0]["legs"][0]
    steps = []
    for step in leg["steps"]:
        if step["travel_mode"] == "TRANSIT":
            t = step["transit_details"]
            steps.append({"type":"transit",
                          "line": t["line"].get("short_name",""),
                          "from": t["departure_stop"]["name"],
                          "to":   t["arrival_stop"]["name"],
                          "dur":  step["duration"]["text"]})
        elif step["travel_mode"] == "WALKING":
            steps.append({"type":"walking",
                          "dist": step["distance"]["text"],
                          "dur":  step["duration"]["text"]})
    return {"total": leg["duration"]["text"], "steps": steps}
# 无 API Key fallback：web_search "{出发地} to {目的地} travel time {城市}"
```

### 2D. 机票（SerpAPI Google Flights）
```python
import urllib.request, json, urllib.parse

def search_flights(dep, arr, date, key, currency="CNY"):
    """申请 SerpAPI Key（免费100次/月）：https://serpapi.com/"""
    params = {"engine":"google_flights","departure_id":dep,"arrival_id":arr,
              "outbound_date":date,"currency":currency,"hl":"zh-cn","api_key":key}
    url = "https://serpapi.com/search?" + urllib.parse.urlencode(params)
    with urllib.request.urlopen(url, timeout=15) as r:
        data = json.loads(r.read())
    results = []
    for f in data.get("best_flights",[])[:5]:
        for seg in f.get("flights",[]):
            results.append({"airline":seg["airline"],
                            "flight":seg["flight_number"],
                            "dep":seg["departure_airport"]["time"],
                            "arr":seg["arrival_airport"]["time"]})
        results[-1]["price"] = f.get("price","N/A")
    return results
# 无 API Key fallback：web_search "{出发城市} to {目的地} flights {日期} price"
```

---

## Phase 3：网络调研

1. brunch café（有咖啡）
2. 多元化晚餐（中餐/日料/泰餐/韩餐/越南菜，每次至少各含 1-2 家）
3. 景点（结合用户偏好：优先经典地标、观景台、历史遗址、百老汇演出）
4. 城市交通（已在 Phase 1 执行则跳过）
5. 天气（超出 16 天预报时）
6. 中高端酒店
7. 机票（无 SerpAPI 时用 web_search）

**餐厅选择规则：**
- brunch 可西式，但正餐必须包含至少 1 餐中餐和 1 餐亚洲料理（日/泰/韩/越）
- 搜索示例：`best Chinese restaurant {城市} 2025` / `best Japanese restaurant {城市} 2025`

---

## Phase 4：生成 Excel（5个Sheet）

读取 `/mnt/skills/public/xlsx/SKILL.md` 后，用 `openpyxl` 生成。

**全局字体规则（所有 Sheet 均适用）：**
```python
# 中文单元格
cell.font = Font(name="宋体", size=10, ...)

# 纯英文单元格（地址、网址、航班号、代码等）
cell.font = Font(name="Times New Roman", size=10, ...)

# 中英文混排单元格：统一使用宋体
cell.font = Font(name="宋体", size=10, ...)
```

**全局行高规则（所有 Sheet 均适用）：**
```python
def est_height(text, col_width, base_h=16, min_h=20):
    if not text: return min_h
    lines = str(text).count('\n') + 1
    chars_per_line = max(1, int(col_width * 1.7))
    wrapped = sum((len(line) // chars_per_line + 1)
                  for line in str(text).split('\n'))
    return max(min_h, wrapped * base_h)

# 每行取所有列中最大估算高度
row_h = max(est_height(cell_val, col_width) for each cell in row)
ws.row_dimensions[r].height = row_h
```

---

### Sheet 1：行程

#### 顶部提示区（3行，无 emoji，纯文字，蓝色背景 #6A9BBC + 白字，宋体）

```
行1：[时差提示] {出发城市} -> {目的地}，时差说明，注意抵达首日安排
行2：[天气提示] 旅行期间天气概况及注意事项
行3：[交通提示] 本次行程推荐出行方式及注意事项
```

#### 列结构（共 8 列，禁止任何 emoji）

| 列 | 名称 | 说明 | 字体 |
|----|------|------|------|
| A | 日期 | 4月1日 | 宋体 |
| B | 星期 | 星期三 | 宋体 |
| C | 天气 | 18C / 9C  晴间多云（无 emoji，无度数符号，用 C） | 宋体 |
| D | 住宿 | 酒店名（宋体）+ 英文地址（Times New Roman）；同一天合并单元格 | 宋体（混排） |
| E | 时间 | 11:00-12:30 | 宋体 |
| F | 行程规划 | 见下方规范 | 宋体 |
| G | 交通方式 | 方式 + 起点 + 终点 + 时长 | 宋体 |
| H | 备注 | 需预订（红字+网址）/ 备选餐厅 | 宋体；网址用 Times New Roman |

#### F列行程规划规范 ⚠️

景点必须写具体打卡路线：
```
[景点名称]
打卡路线：[起点] -> [打卡点1（说明）] -> [打卡点2（说明）] -> [终点]
```

示例：
```
哈佛大学
打卡路线：Harvard Square（T出站）-> Harvard Book Store -> Massachusetts Hall（最古老建筑）
          -> John Harvard铜像（传说摸左脚带来好运）-> Widener Library台阶 -> Harvard Yard草坪
```

餐厅/咖啡馆注明类型：`午餐：[名称]` / `咖啡：[名称]` / `晚餐：[名称]`

#### 每日结构

```
10:00-11:00   起床、化妆出发准备
11:00-12:30   午餐/咖啡：[餐厅名]
[景点1（含打卡路线）]
[景点2（可选，含打卡路线）]
晚餐：[餐厅名]
[晚间活动（如百老汇、夜景观景台）]
回酒店
```

#### 配色（Anthropic brand-guidelines）

| 元素 | 颜色 | 说明 |
|------|------|------|
| 表头行 | #141413（深黑）+ 白字 | brand dark |
| 提示区（顶部） | #6A9BBC（蓝）+ 白字 | brand blue |
| 城市分隔行 | #788C5D（绿）+ 白字 | brand green |
| 数据奇数行 | #FAF9F5（米白）| brand light |
| 数据偶数行 | #E8E6DC（浅灰）| brand light gray |
| 需预订红字 | #FF0000 | 警示 |

列宽：A=10, B=7, C=18, D=26, E=13, F=38, G=28, H=32
行高：自适应（用 `est_height()` 计算，不使用固定值）

---

### Sheet 2：住宿推荐

- 每个城市 3 家中高端酒店
- 价格实时汇率转 CNY，注明汇率来源和日期
- 表头色：brand green #788C5D；宋体加粗白字
- 城市用分隔标题行区分（#6A9BBC 蓝色）
- 行高自适应

---

### Sheet 3：机票参考

- 去程 ± 3 天 + 城市间中段（如 Amtrak）+ 回程 ± 2 天
- 计划日期标黄高亮（#FFF9C4）
- 表头色：brand green #788C5D；段落标题：#6A9BBC
- 无 emoji；行高自适应

---

### Sheet 4：物品清单

根据以下因素动态生成：
1. **出发城市**：若在中国大陆，需要签证+转换插头；若在美国境内，两者均不需要提醒
2. **目的地国家**：插头类型、货币、交通 App
3. **季节**：防晒/保暖/雨具
4. **行程特点**：自驾（国际驾照）、海岛、徒步等

三色分区（宋体，无 emoji）：
- 必备：浅橙色 #FFE8DF
- 目的地特需：浅灰色 #E8E6DC
- 可选：米白色 #FAF9F5

行高自适应。

---

### Sheet 5：景点介绍

**纯文字，不插入任何图片。**

为行程中的每一个重要景点（通常 5-10 个）撰写详细介绍。

**结构（每个景点）：**
1. **景点标题行**（合并 A-C，brand green #788C5D 背景白字，宋体 11pt 加粗）
2. **基本信息表格**（B列为标签宋体加粗，C列为内容宋体，A列同色留空）：
   - 地址（英文地址用 Times New Roman）
   - 开放时间
   - 推荐游览时长
   - 票价/购票方式
   - 具体打卡路线（与行程 F 列保持一致）
3. **历史背景标题行**（合并 A-C，#141413 深黑背景白字，宋体 10pt 加粗）
4. **历史正文**（合并 A-C，#FAF9F5 米白背景，宋体 9pt）：
   - 历史地标：建造年代、历史事件、重要意义
   - 博物馆：镇馆之宝、著名展品和创作者
   - 观景台：建筑特色、最佳观看角度
   - 拍照贴士：最佳拍摄位置、光线建议

**配色：**
- 景点标题：#788C5D（绿）+ 白字
- 基本信息行：#E8E6DC（浅灰）
- 历史标题：#141413（深黑）+ 白字
- 历史正文：#FAF9F5（米白）

**行高规则：**
- 景点标题行：26
- 基本信息各行：用 `est_height(value, col_C_width)` 自适应
- 历史正文行：根据段落数和字符总量估算，`min(h_content, 600)`

列宽：A=20, B=16, C=64

---

## Phase 5：输出

保存至 `/mnt/user-data/outputs/{目的地}_{天数}天旅行计划.xlsx`，用 `present_files` 发送。
一句话说明即可。