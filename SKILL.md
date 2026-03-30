---
name: weather-card
description: >
  Generate weather cards via an agent-first workflow. Supports gathering weather facts, normalizing them,
  and generating visual weather cards in app/social/xiaohongshu styles via Gemini.
  Use when user asks for "天气卡片", "今日天气卡", "城市天气海报", "weather card", "小红书天气卡", or "社媒天气图".
---

# Weather Card Skill

Generate shareable weather cards using an agent-orchestrated workflow.
Gather weather facts first, then use Gemini to generate the final image.

## Prerequisites

- `gemini-web-cli` installed: `npm install -g @lumiaqian/gemini-web-cli`
- Google account logged in (first use)

## Workflow

### Step 1: Resolve Inputs

Identify:

- `city`
- `date`
- `style`

Defaults:

- `date`: today
- `style`: infer from user intent, otherwise `social`

If `city` is missing, ask the user before continuing.

### Step 2: Get Weather Data

Do not ask Gemini to decide the weather directly.

Use agents to gather weather information first, then normalize it into this structure:

| Field | Example |
|--------|---------|
| city | 北京 |
| date | 2026-03-29 |
| weather | 晴 |
| temp_high | 18°C |
| temp_low | 8°C |
| humidity | 45% |
| wind | 3级 |
| rain_prob | 10% |
| sunrise | 06:15 |
| sunset | 18:42 |
| source | multi-source |
| confidence | high |

Requirements:

- fill all core weather fields before image generation
- if sources conflict on important values, lower confidence
- if key fields are missing, retry fact gathering before continuing

Gemini may help rewrite or summarize structured data later, but it should not be the primary weather source.

### Step 3: Select Style

Based on user intent:

| Intent | Style |
|--------|-------|
| "信息清楚" / "app" | **app** — Clean hierarchy, data-focused |
| "社媒卡片" / "social" | **social** — Soft colors, title-focused |
| "小红书风" / "博主风" | **xiaohongshu** — Lifestyle tone, pastel colors |

If the user explicitly names a style, use it.

### Step 4: Generate Copy

After weather data is validated, optionally use Gemini to generate:

- a short title
- a subtitle
- a lifestyle suggestion

Do not let Gemini invent missing weather values.

### Step 5: Generate Image

```bash
gemini-web-cli generate-image \
  --prompt "[see prompt template below]" \
  --timeout-ms 180000 \
  --json
```

Prompt rules:

- always use validated weather fields
- include city, date, weather, and temperature
- include style-specific composition guidance
- never ask Gemini to guess the weather

## Prompt Templates

### App Style

```
Create a weather card image with these specifications:

**Layout**: Clean app-style card, 16:9 aspect ratio

**Content**:
- City: {city}
- Date: {date}
- Weather icon: {weather_symbol}
- Temperature: {temp_high} / {temp_low}
- Humidity: {humidity}
- Wind: {wind}
- Rain probability: {rain_prob}

**Style Requirements**:
- Modern weather app aesthetic
- Clear information hierarchy
- Minimal decoration, data takes priority
- Professional color scheme
- Sans-serif typography
- Show weather-appropriate background

**Composition**:
- Top: City name and date
- Center: Large weather icon with temperature
- Bottom: Weather details in clean grid
```

### Social Style

```
Create a shareable social media weather card:

**Layout**: Social card format, 4:3 aspect ratio

**Content**:
- City: {city}
- Date: {date}
- Weather: {weather}
- Temperature: {temp_high} / {temp_low}

**Style Requirements**:
- Eye-catching, title-focused design
- Soft, warm color palette
- Romantic/moody atmosphere
- Lifestyle feel, not just weather report
- Decorative but readable

**Example Text Elements**:
- Tagline: "今日天气 · {city}"
- Subtitle: "{city}，{short_mood_line}"

**Composition**:
- Large atmospheric background
- Overlaid title text
- Temperature as hero number
- Small weather icons at bottom
```

### Xiaohongshu Style

```
Create a Xiaohongshu (Little Red Book) style weather card:

**Layout**: Portrait 3:4, mobile-friendly

**Content**:
- City: {city}
- Date: {date}
- Weather: {weather}
- Temperature: {temp_high} / {temp_low}

**Style Requirements**:
- Soft, creamy pastel colors
- Cute sticker-like labels
- Lifestyle blogger tone
- Playful weather icons
- Chinese aesthetic with modern touch

**Text Elements**:
- Label: "今日天气"
- Title: "{city}天气"
- Suggestion: "宜：{lifestyle_suggestion}"
- Temperature as prominent display

**Composition**:
- Soft gradient background
- Main weather illustration
- Sticker-style weather tags
- Lifestyle suggestion badge
- Date in cute format
```

## Lifestyle Suggestions

| Weather | Suggestion |
|--------|------------|
| 晴 | 户外散步、拍照打卡 |
| 多云 | 出门踏青、野餐 |
| 阴 | 室内咖啡、读书时光 |
| 雨 | 在家听雨、泡杯热茶 |
| 雪 | 出门玩雪、喝热巧克力 |

## Weather Symbols

| Weather | Emoji | Description |
|---------|-------|-------------|
| 晴 | ☀️ | Sunny, clear sky |
| 多云 | ⛅ | Partly cloudy |
| 阴 | ☁️ | Cloudy |
| 小雨 | 🌧️ | Light rain |
| 中雨 | 🌧️ | Moderate rain |
| 雷阵雨 | ⛈️ | Thunderstorm |
| 雪 | ❄️ | Snow |
| 大风 | 💨 | Windy |

## Output

Save the generated image and provide:

1. **Image file**: `{city}-{date}-weather-card.{ext}`
2. **Data summary**: Weather data in markdown format

Suggested markdown format:

```md
# 天气数据

- 城市：{city}
- 日期：{date}
- 天气：{weather}
- 温度：{temp_low} ~ {temp_high}
- 湿度：{humidity}
- 风力：{wind}
- 降雨概率：{rain_prob}
- 日出：{sunrise}
- 日落：{sunset}
- 来源：{source}
- 置信度：{confidence}
```

## Error Handling

| Error | Solution |
|-------|----------|
| City missing | Ask user for city |
| Incomplete weather data | Retry fact gathering before image generation |
| Conflicting weather values | Lower confidence and report discrepancy |
| "Not logged in" | Run `gemini-web-cli check-login --json` first |
| Timeout | Increase `--timeout-ms 300000` |
| Element not found | Run `gemini-web-cli reload-page --json` then retry |

## Notes

- Gather weather facts first, generate image second
- Use Gemini for wording and image generation, not as the primary weather source
- Keep prompts concise but include all validated weather data
- If generation fails, retry once before reporting
