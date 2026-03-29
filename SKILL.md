---
name: weather-card
description: >
  Generate weather cards via Gemini. Supports fetching weather data, generating visual weather cards in app/social/xiaohongshu styles.
  Use when user asks for "天气卡片", "今日天气卡", "城市天气海报", "weather card", "小红书天气卡", or "社媒天气图".
---

# Weather Card Skill

Generate shareable weather cards using Gemini. Fetches real weather data and generates beautiful images.

## Prerequisites

- `gemini-web-cli` installed: `npm install -g @lumiaqian/gemini-web-cli`
- Google account logged in (first use)

## Workflow

### Step 1: Get Weather Data

```bash
gemini-web-cli send-message \
  --message "请用中文简洁回答：北京今天（YYYY-MM-DD）的天气，包括：天气现象、温度（高/低）、湿度、风速、降雨概率、日出日落时间" \
  --json
```

Extract and structure the data:

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

### Step 2: Select Style

Based on user intent:

| Intent | Style |
|--------|-------|
| "信息清楚" / "app" | **app** — Clean hierarchy, data-focused |
| "社媒卡片" / "social" | **social** — Soft colors, title-focused |
| "小红书风" / "博主风" | **xiaohongshu** — Lifestyle tone, pastel colors |

### Step 3: Generate Image

```bash
gemini-web-cli generate-image \
  --prompt "[see prompt template below]" \
  --timeout-ms 180000 \
  --json
```

## Prompt Templates

### App Style

```
Create a weather card image with these specifications:

**Layout**: Clean app-style card, 16:9 aspect ratio

**Content**:
- City: {city}
- Date: {date}
- Weather icon: {weather_symbol}
- Temperature: {temp_high}° / {temp_low}°
- Humidity: {humidity}
- Wind: {wind}
- Rain probability: {rain_prob}%

**Style Requirements**:
- Modern weather app aesthetic
- Clear information hierarchy
- Minimal decoration, data takes priority
- Professional color scheme (blues, whites, grays)
- Sans-serif typography
- Show weather-appropriate background (sunny = blue sky, rainy = gray with rain drops, etc.)

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
- Temperature: {temp_high}° / {temp_low}°

**Style Requirements**:
- Eye-catching, title-focused design
- Soft, warm color palette
- Romantic/moody atmosphere
- Text as a headline, not just data
- Add decorative elements (clouds, sun rays, etc.)
- Lifestyle feel, not just weather report

**Example Text Elements**:
- Tagline: "今日天气 · {city}"
- Subtitle: "[季节感]的{city}，宜{activity}"

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
- Temperature: {temp_high}° / {temp_low}°

**Style Requirements**:
- Soft, creamy pastel colors
- Cute sticker-like labels
- Lifestyle blogger tone
- Warmer, more emotional than social style
- Playful weather icons
- Chinese aesthetic with modern touch

**Text Elements**:
- Label: "今日天气"
- Title: "[诗意/场景化描述], {city}"
- Suggestion: "宜：{lifestyle_suggestion}"
- Temperature as prominent display

**Composition**:
- Soft gradient background (dawn/day/dusk based on time)
- Main weather illustration
- Sticker-style weather tags
- Lifestyle suggestion badge
- Date in cute format
```

## Lifestyle Suggestions (for xiaohongshu style)

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

## Error Handling

| Error | Solution |
|-------|----------|
| "Not logged in" | Run `gemini-web-cli check-login --json` first |
| Timeout | Increase `--timeout-ms 300000` |
| Element not found | Run `gemini-web-cli reload-page --json` then retry |

## Notes

- Gemini image generation takes 60-180 seconds
- Always use `--timeout-ms 180000` minimum
- If generation fails, retry once before reporting
- Keep prompts concise but include all weather data
