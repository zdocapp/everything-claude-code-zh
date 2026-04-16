---
name: video-editing
description: AI辅助的视频编辑工作流程，用于剪辑、结构化和增强实拍素材。涵盖从原始拍摄到FFmpeg、Remotion、ElevenLabs、fal.ai的完整流程，以及最终在Descript或CapCut中的润色。适用于用户想要编辑视频、剪辑素材、创建视频博客或制作视频内容时。
origin: ECC
---

# 视频编辑

AI辅助的真实素材编辑。非根据提示生成。快速编辑现有视频。

## 何时激活

* 用户想要编辑、剪辑或组织视频素材
* 将长录制内容转化为短视频内容
* 从原始拍摄素材构建vlog、教程或演示视频
* 为现有视频添加叠加层、字幕、音乐或画外音
* 为不同平台（YouTube、TikTok、Instagram）重新构图视频
* 用户提及"编辑视频"、"剪辑这段素材"、"制作vlog"或"视频工作流"

## 核心理念

当你不再要求AI创建整个视频，而是开始用它来压缩、组织和增强真实素材时，AI视频编辑才真正有用。其价值不在于生成，而在于压缩。

## 工作流程

```
Screen Studio / 原始素材
  → Claude / Codex
  → FFmpeg
  → Remotion
  → ElevenLabs / fal.ai
  → Descript 或 CapCut
```

每一层都有特定的任务。不要跳过任何层。不要试图让一个工具完成所有事情。

## 第1层：采集（Screen Studio / 原始素材）

收集源材料：

* **Screen Studio**：用于应用演示、编码会话、浏览器工作流的精美屏幕录制
* **原始摄像机素材**：vlog素材、访谈、活动录制
* **通过VideoDB进行桌面捕获**：带有实时上下文的会话录制（参见 `videodb` 技能）

输出：准备就绪的原始文件，等待组织。

## 第2层：组织（Claude / Codex）

使用Claude Code或Codex来：

* **转录和标记**：生成转录稿，识别主题和要点
* **规划结构**：决定保留什么、剪掉什么、何种顺序有效
* **识别无效片段**：找出停顿、跑题、重复拍摄的部分
* **生成剪辑决策列表**：剪辑的时间戳、需要保留的片段
* **搭建FFmpeg和Remotion代码框架**：生成命令和合成代码

```
示例提示：
"这是一份4小时录音的文字记录。请找出最适合制作24分钟vlog的8个最强片段。
为每个片段提供FFmpeg剪辑命令。"
```

这一层关乎结构，而非最终的创意品味。

## 第3层：确定性剪辑（FFmpeg）

FFmpeg处理枯燥但关键的工作：分割、修剪、拼接和预处理。

### 按时间戳提取片段

```bash
ffmpeg -i raw.mp4 -ss 00:12:30 -to 00:15:45 -c copy segment_01.mp4
```

### 根据剪辑决策列表批量剪辑

```bash
#!/bin/bash
# cuts.txt: start,end,label
while IFS=, read -r start end label; do
  ffmpeg -i raw.mp4 -ss "$start" -to "$end" -c copy "segments/${label}.mp4"
done < cuts.txt
```

### 拼接片段

```bash
# Create file list
for f in segments/*.mp4; do echo "file '$f'"; done > concat.txt
ffmpeg -f concat -safe 0 -i concat.txt -c copy assembled.mp4
```

### 创建代理文件以加速编辑

```bash
ffmpeg -i raw.mp4 -vf "scale=960:-2" -c:v libx264 -preset ultrafast -crf 28 proxy.mp4
```

### 提取音频用于转录

```bash
ffmpeg -i raw.mp4 -vn -acodec pcm_s16le -ar 16000 audio.wav
```

### 标准化音频电平

```bash
ffmpeg -i segment.mp4 -af loudnorm=I=-16:TP=-1.5:LRA=11 -c:v copy normalized.mp4
```

## 第4层：可编程合成（Remotion）

Remotion将编辑问题转化为可组合的代码。用它来处理传统编辑器令人头疼的事情：

### 何时使用Remotion

* 叠加层：文本、图像、品牌标识、下三分之一字幕
* 数据可视化：图表、统计数据、动画数字
* 动态图形：转场、解说动画
* 可组合场景：跨视频可复用的模板
* 产品演示：带注释的截图、UI高亮

### 基本的Remotion合成

```tsx
import { AbsoluteFill, Sequence, Video, useCurrentFrame } from "remotion";

export const VlogComposition: React.FC = () => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill>
      {/* Main footage */}
      <Sequence from={0} durationInFrames={300}>
        <Video src="/segments/intro.mp4" />
      </Sequence>

      {/* Title overlay */}
      <Sequence from={30} durationInFrames={90}>
        <AbsoluteFill style={{
          justifyContent: "center",
          alignItems: "center",
        }}>
          <h1 style={{
            fontSize: 72,
            color: "white",
            textShadow: "2px 2px 8px rgba(0,0,0,0.8)",
          }}>
            The AI Editing Stack
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* Next segment */}
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

详细模式和API参考请参阅 [Remotion文档](https://www.remotion.dev/docs)。

## 第5层：生成资产（ElevenLabs / fal.ai）

只生成你需要的部分。不要生成整个视频。

### 使用ElevenLabs生成画外音

```python
import os
import requests

resp = requests.post(
    f"https://api.elevenlabs.io/v1/text-to-speech/{voice_id}",
    headers={
        "xi-api-key": os.environ["ELEVENLABS_API_KEY"],
        "Content-Type": "application/json"
    },
    json={
        "text": "Your narration text here",
        "model_id": "eleven_turbo_v2_5",
        "voice_settings": {"stability": 0.5, "similarity_boost": 0.75}
    }
)
with open("voiceover.mp3", "wb") as f:
    f.write(resp.content)
```

### 使用fal.ai生成音乐和音效

使用 `fal-ai-media` 技能处理：

* 背景音乐生成
* 音效（使用ThinkSound模型进行视频到音频转换）
* 转场音效

### 使用fal.ai生成视觉效果

用于生成不存在的插入镜头、缩略图或B-roll素材：

```
generate(app_id: "fal-ai/nano-banana-pro", input_data: {
  "prompt": "科技视频博客的专业缩略图，深色背景，屏幕上有代码",
  "image_size": "landscape_16_9"
})
```

### VideoDB生成式音频

如果配置了VideoDB：

```python
voiceover = coll.generate_voice(text="Narration here", voice="alloy")
music = coll.generate_music(prompt="lo-fi background for coding vlog", duration=120)
sfx = coll.generate_sound_effect(prompt="subtle whoosh transition")
```

## 第6层：最终润色（Descript / CapCut）

最后一层是人工操作。使用传统编辑器进行：

* **节奏调整**：调整感觉太快或太慢的剪辑点
* **字幕**：自动生成，然后手动清理
* **色彩分级**：基本校正和氛围调整
* **最终音频混音**：平衡人声、音乐和音效的电平
* **导出**：平台特定的格式和质量设置

品味体现在这一层。AI负责清理重复性工作。你来做最终的创意决策。

## 社交媒体重新构图

不同平台需要不同的宽高比：

| 平台 | 宽高比 | 分辨率 |
|----------|-------------|------------|
| YouTube | 16:9 | 1920x1080 |
| TikTok / Reels | 9:16 | 1080x1920 |
| Instagram Feed | 1:1 | 1080x1080 |
| X / Twitter | 16:9 或 1:1 | 1280x720 或 720x720 |

### 使用FFmpeg重新构图

```bash
# 16:9 to 9:16 (center crop)
ffmpeg -i input.mp4 -vf "crop=ih*9/16:ih,scale=1080:1920" vertical.mp4

# 16:9 to 1:1 (center crop)
ffmpeg -i input.mp4 -vf "crop=ih:ih,scale=1080:1080" square.mp4
```

### 使用VideoDB重新构图

```python
from videodb import ReframeMode

# Smart reframe (AI-guided subject tracking)
reframed = video.reframe(start=0, end=60, target="vertical", mode=ReframeMode.smart)
```

## 场景检测与自动剪辑

### FFmpeg场景检测

```bash
# Detect scene changes (threshold 0.3 = moderate sensitivity)
ffmpeg -i input.mp4 -vf "select='gt(scene,0.3)',showinfo" -vsync vfr -f null - 2>&1 | grep showinfo
```

### 用于自动剪辑的静音检测

```bash
# Find silent segments (useful for cutting dead air)
ffmpeg -i input.mp4 -af silencedetect=noise=-30dB:d=2 -f null - 2>&1 | grep silence
```

### 高光片段提取

使用Claude分析转录稿和场景时间戳：

```
"给定这份带时间戳的转录文本和这些场景切换点，
为社交媒体识别出5个最具吸引力的30秒片段。"
```

## 各工具最擅长之处

| 工具 | 优势 | 劣势 |
|------|----------|----------|
| Claude / Codex | 组织、规划、代码生成 | 不是创意品味层 |
| FFmpeg | 确定性剪辑、批量处理、格式转换 | 没有可视化编辑界面 |
| Remotion | 可编程叠加层、可组合场景、可复用模板 | 对非开发者有学习曲线 |
| Screen Studio | 即时获得精美的屏幕录制 | 仅限屏幕捕获 |
| ElevenLabs | 语音、旁白、音乐、音效 | 不是工作流的核心 |
| Descript / CapCut | 最终节奏调整、字幕、润色 | 手动操作，无法自动化 |

## 关键原则

1. **编辑，而非生成。** 此工作流用于剪辑真实素材，而非根据提示创建。
2. **结构先于风格。** 在第2层先把故事结构理清，再处理视觉内容。
3. **FFmpeg是骨干。** 枯燥但关键。长素材在此变得易于管理。
4. **Remotion用于可重复性。** 如果你会做不止一次，就把它做成Remotion组件。
5. **有选择地生成。** 仅对不存在的资产使用AI生成，而非所有内容。
6. **品味是最后一层。** AI负责清理重复性工作。你来做最终的创意决策。

## 相关技能

* `fal-ai-media` — AI图像、视频和音频生成
* `videodb` — 服务器端视频处理、索引和流式传输
* `content-engine` — 平台原生内容分发
