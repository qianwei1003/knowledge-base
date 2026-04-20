# ECC 视频编辑工作流

> 来源：ECC `video-editing` 技能 | 提炼时间：2026-04-20

---

## 目录

- [核心论点](#核心论点)
- [AI 视频编辑六层管道](#ai-视频编辑六层管道)
- [Layer 1: 采集](#layer-1-采集)
- [Layer 2: 组织（Claude / Codex）](#layer-2-组织claude--codex)
- [Layer 3: 确定性剪辑（FFmpeg）](#layer-3-确定性剪辑ffmpeg)
- [Layer 4: 程序化合成（Remotion）](#layer-4-程序化合成remotion)
- [Layer 5: 生成资产（ElevenLabs / fal.ai）](#layer-5-生成资产elevenlabs--flaiai)
- [Layer 6: 最终打磨（Descript / CapCut）](#layer-6-最终打磨descript--capcut)
- [社交媒体重新构图](#社交媒体重新构图)
- [场景检测与自动剪辑](#场景检测与自动剪辑)
- [工具优选表](#工具优选表)

---

## 核心论点

AI 视频编辑的价值在于**压缩**，而非生成。当停止让 AI 创建整个视频，开始用它压缩、组织和增强真实素材时，AI 视频编辑才有价值。

## 关键词

"edit video"、"cut this footage"、"make a vlog"、"video workflow"

---

## AI 视频编辑六层管道

```
Screen Studio / 原始素材
  → Claude / Codex         # 组织、规划、代码生成
  → FFmpeg                 # 确定性剪辑、批量处理、格式转换
  → Remotion               # 可编程叠加、合成场景、可复用模板
  → ElevenLabs / fal.ai    # 配音、音乐、音效
  → Descript 或 CapCut     # 最终润色
```

每层工具有明确的职责。不要跳过层次。不要让一个工具做所有事情。

---

## Layer 1: 采集

收集源材料：

- **Screen Studio**：面向 App 演示、编码会话、浏览器工作流的精致屏幕录制
- **原始摄像机素材**：Vlog、访谈、活动录制
- **桌面捕获（VideoDB）**：带实时上下文的会话录制

输出：准备好整理的原始文件。

---

## Layer 2: 组织（Claude / Codex）

使用 Claude Code 或 Codex：

- **转录和标注**：生成转录文本，识别主题和要点
- **规划结构**：决定保留什么、剪掉什么、什么顺序有效
- **识别无效片段**：找出停顿、跑题、重复镜头
- **生成剪辑决策列表**：剪切时间戳、保留片段
- **生成 FFmpeg 和 Remotion 代码**：生成命令和合成文件

```
示例提示：
"这是一段 4 小时录制的转录文本。找出制作 24 分钟 Vlog 的 8 个最强片段。
给出每个片段的 FFmpeg 剪切命令。"
```

---

## Layer 3: 确定性剪辑（FFmpeg）

FFmpeg 处理无聊但关键的工作：分割、修剪、合并和预处理。

### 按时间戳提取片段

```bash
ffmpeg -i raw.mp4 -ss 00:12:30 -to 00:15:45 -c copy segment_01.mp4
```

### 批量剪切

```bash
# cuts.txt: start,end,label
while IFS=, read -r start end label; do
  ffmpeg -i raw.mp4 -ss "$start" -to "$end" -c copy "segments/${label}.mp4"
done < cuts.txt
```

### 合并片段

```bash
for f in segments/*.mp4; do echo "file '$f'"; done > concat.txt
ffmpeg -f concat -safe 0 -i concat.txt -c copy assembled.mp4
```

### 创建代理以加快编辑速度

```bash
ffmpeg -i raw.mp4 -vf "scale=960:-2" -c:v libx264 -preset ultrafast -crf 28 proxy.mp4
```

### 提取音频用于转录

```bash
ffmpeg -i raw.mp4 -vn -acodec pcm_s16le -ar 16000 audio.wav
```

### 音频标准化

```bash
ffmpeg -i segment.mp4 -af loudnorm=I=-16:TP=-1.5:LRA=11 -c:v copy normalized.mp4
```

---

## Layer 4: 程序化合成（Remotion）

Remotion 将编辑问题转化为可组合的代码。用于传统编辑器难以处理的任务：

- **叠加层**：文字、图片、品牌标识、下三分字幕
- **数据可视化**：图表、统计数据、动态数字
- **动态图形**：转场、解释性动画
- **可组合场景**：跨视频的可复用模板
- **产品演示**：带标注的截图、UI 高亮

### 何时使用 Remotion

- 叠加层：文字、图片、品牌标识、下三分字幕
- 数据可视化：图表、统计数据、动态数字
- 动态图形：转场、解释性动画
- 可组合场景：跨视频的可复用模板
- 产品演示：带标注的截图、UI 高亮

### 基础 Remotion 合成

```tsx
import { AbsoluteFill, Sequence, Video, useCurrentFrame } from "remotion";

export const VlogComposition: React.FC = () => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill>
      {/* 主要素材 */}
      <Sequence from={0} durationInFrames={300}>
        <Video src="/segments/intro.mp4" />
      </Sequence>

      {/* 标题叠加 */}
      <Sequence from={30} durationInFrames={90}>
        <AbsoluteFill style={{ justifyContent: "center", alignItems: "center" }}>
          <h1 style={{ fontSize: 72, color: "white", textShadow: "2px 2px 8px rgba(0,0,0,0.8)" }}>
            The AI Editing Stack
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* 下一片段 */}
      <Sequence from={300} durationInFrames={450}>
        <Video src="/segments/demo.mp4" />
      </Sequence>
    </AbsoluteFill>
  );
};
```

### 渲染输出

```bash
npx remotion render src/index.ts VlogComposition output.mp4
```

---

## Layer 5: 生成资产（ElevenLabs / fal.ai）

按需生成，不要生成整个视频。

### fal.ai 图像生成

```
# Nano Banana 2（快速）
generate(
  model_name: "fal-ai/nano-banana-2",
  input: {
    "prompt": "futuristic cityscape at sunset, cyberpunk style",
    "image_size": "landscape_16_9",
    "num_images": 1,
    "seed": 42
  }
)

# Nano Banana Pro（高保真）
generate(
  model_name: "fal-ai/nano-banana-pro",
  input: {
    "prompt": "professional product photo of wireless headphones on marble surface",
    "image_size": "square",
    "num_images": 1,
    "guidance_scale": 7.5
  }
)
```

### fal.ai 视频生成

```
# Seedance 1.0 Pro（高运动质量）
generate(
  model_name: "fal-ai/seedance-1-0-pro",
  input: {
    "prompt": "a drone flyover of a mountain lake at golden hour",
    "duration": "5s",
    "aspect_ratio": "16:9",
    "seed": 42
  }
)

# Kling Video v3 Pro（原生音频生成）
generate(
  model_name: "fal-ai/kling-video/v3/pro",
  input: {
    "prompt": "ocean waves crashing on a rocky coast",
    "duration": "5s",
    "aspect_ratio": "16:9"
  }
)
```

### ElevenLabs 配音

```python
import os, requests

resp = requests.post(
    f"https://api.elevenlabs.io/v1/text-to-speech/{voice_id}",
    headers={"xi-api-key": os.environ["ELEVENLABS_API_KEY"], "Content-Type": "application/json"},
    json={
        "text": "Your narration text here",
        "model_id": "eleven_turbo_v2_5",
        "voice_settings": {"stability": 0.5, "similarity_boost": 0.75}
    }
)
with open("voiceover.mp3", "wb") as f:
    f.write(resp.content)
```

---

## Layer 6: 最终打磨（Descript / CapCut）

最后一层是人类编辑：

- **节奏**：调整感觉太快或太慢的剪辑
- **字幕**：自动生成后手动清理
- **调色**：基本校正和氛围
- **混音**：平衡人声、音乐和音效
- **导出**：平台特定格式和质量设置

这里是品味所在。AI 清除重复性工作。你做最终决定。

---

## 社交媒体重新构图

| 平台 | 宽高比 | 分辨率 |
|------|--------|--------|
| YouTube | 16:9 | 1920x1080 |
| TikTok / Reels | 9:16 | 1080x1920 |
| Instagram Feed | 1:1 | 1080x1080 |
| X / Twitter | 16:9 或 1:1 | 1280x720 或 720x720 |

### FFmpeg 重新构图

```bash
# 16:9 → 9:16（中心裁切）
ffmpeg -i input.mp4 -vf "crop=ih*9/16:ih,scale=1080:1920" vertical.mp4

# 16:9 → 1:1（中心裁切）
ffmpeg -i input.mp4 -vf "crop=ih:ih,scale=1080:1080" square.mp4
```

---

## 场景检测与自动剪辑

### 场景变化检测

```bash
ffmpeg -i input.mp4 -vf "select='gt(scene,0.3)',showinfo" -vsync vfr -f null - 2>&1 | grep showinfo
```

### 静音检测

```bash
ffmpeg -i input.mp4 -af silencedetect=noise=-30dB:d=2 -f null - 2>&1 | grep silence
```

---

## 工具优选表

| 工具 | 优势 | 劣势 |
|------|------|------|
| Claude / Codex | 组织、规划、代码生成 | 不是创意品味层 |
| FFmpeg | 确定性剪辑、批量处理、格式转换 | 无可视化编辑 UI |
| Remotion | 可编程叠加、可组合场景、可复用模板 | 非开发者有学习曲线 |
| Screen Studio | 即时精致屏幕录制 | 只能屏幕捕获 |
| ElevenLabs | 配音、旁白、音乐、音效 | 不是工作流中心 |
| Descript / CapCut | 最终节奏、字幕、润色 | 手动，不可自动化 |

---

**记住**：编辑，不要生成。这个工作流用于剪切真实素材，不是从提示词创建视频。
