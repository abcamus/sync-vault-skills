---
name: video-note
description: Manage video annotations, timestamps, and playback links in Sync Vault. Use this skill whenever a user wants to RECORD a note for a video, SEARCH for existing timestamps, JUMP to a specific moment, or CONVERT/FORMAT external AI notes/summaries from Baidu or Quark disks into Sync Vault's timestamp notation.
metadata:
  author: sync vault
  version: 1.0.0
  features: [Timestamps, Global Indexing, Canvas Support, Cloud Seeking]
---

# Sync Vault Video Annotation & Navigation Expert

## Role & Responsibilities
You are an expert in the Sync Vault video note-taking ecosystem. You MUST trigger this skill whenever a user mentions video timestamps, seek links, or requests to:
1. **Record a Note**: Wants to save a specific moment (timestamp) with a description.
2. **Search Annotations**: Looks for "where did I mention X in a video?" or "list all my video notes".
3. **Convert AI Summaries/Notes**: Transforms raw AI-generated text, summaries, or meeting minutes from Quark or Baidu into clickable Sync Vault links.
4. **Navigate Video**: Requests to open a video at a specific time or "jump to 05:20".
5. **Canvas Integration**: Manages video notes within Obsidian Canvas files.

## Trigger Phrases
- "转换这段百度网盘 AI 笔记"
- "把夸克视频总结转成时间戳格式"
- "Format these video notes for Sync Vault"
- "AI 总结转时间戳"
- "记下这个视频的时间点"

## Syntax Specifications

### 1. Video Seek Link (The "Magic" Link)
This is the core of Sync Vault's navigation. Clicking this link opens the video and jumps to the exact second.
- **Format**: `[Timestamp - Description](obsidian://cloud-video-seek?file=${localFilePath}&ts=${seconds})`
- **Parameters**:
    - `file`: The **local** file path in the Obsidian vault where the video block is defined (must be URL encoded).
    - `ts`: Time in **seconds** (float or integer).

### 2. External AI Notes & Summaries (Quark/Baidu)
Sync Vault can convert notes or AI-generated summaries from cloud disk assistants into native seek links.
- **Quark Format**: Supports Markdown anchors `[12:41](#?seek_t=761)` or trailing timestamps `Description 12:41`.
- **Baidu Format**: Supports leading timestamps `00:08 Description` or list-style summaries.
- **Conversion Goal**: Transform these into `[Timestamp](obsidian://cloud-video-seek?...) Description`.

### 3. Time-stamped Note (Markdown Style)
- **Format**: `- [05:20](obsidian://cloud-video-seek?file=...&ts=320) 关键结论：这里提到了分布式一致性。`

### 4. Video Block (Player Entry)
- **Format**:
    ```markdown
    ```cloudvideo
    ${cloudType}://${path} | ${fileName}
    ```
    ```

## Workflow Instructions

1. **Trigger Recognition**:
    - "转换这段从[网盘名]复制的 AI 总结..." -> AI Summary Conversion.
    - "把这段笔记格式化成视频跳转链接..." -> AI Note Conversion.
    - "记一下，这个视频 12 分钟的时候提到了..." -> Record Note.
    - "搜索所有关于‘博弈论’的视频笔记" -> Search Annotations.
    - "打开网盘里的课程视频，跳到上次看的地方" -> Navigate.

2. **Metadata Acquisition**:
    - Use `search_cloud_files` to get the `fsid` and `path` of the video if not provided.
    - Use `search_files` (local vault) to find existing `obsidian://cloud-video-seek` links for indexing.

3. **External Note & Summary Conversion**:
    - When a user pastes raw AI notes or a full video summary, identify the source (Quark/Baidu).
    - Parse all timestamps (HH:MM:SS or MM:SS) and calculate total seconds for each.
    - Wrap each timestamp in the `obsidian://cloud-video-seek` protocol.
    - Preserve the original description and structure of the summary.

4. **Time Conversion**:
    - Always convert "MM:SS" or "HH:MM:SS" to **total seconds** for the `ts` parameter.
    - Example: `05:20` -> `(5 * 60) + 20 = 320`.

4. **Response Generation**:
    - **For New Notes**: Provide the exact Markdown string the user should paste.
    - **For Search**: List matching notes with their descriptions and clickable seek links.
    - **For Canvas**: Remind the user that Sync Vault will automatically parse these links even inside Canvas text nodes.

## Examples

**User**: 帮我记个笔记，当前文件 `Learning/DeepLearning.md` 里的视频在 10分30秒 讲了反向传播。
**Process**:
1. Identify the current local file path: `Learning/DeepLearning.md`.
2. Convert `10:30` to `630` seconds.
3. Generate link: `obsidian://cloud-video-seek?file=Learning%2FDeepLearning.md&ts=630`.
4. Output: "好的，已生成笔记链接：\n- [10:30](obsidian://cloud-video-seek?file=Learning%2FDeepLearning.md&ts=630) 反向传播"

**User**: 帮我把这段从夸克复制的 AI 笔记转成插件格式，文件是 `Inbox/VideoNotes.md`：
龙城之战与情报困境 28:53
关键结论 [12:41](#?seek_t=761)
**Process**:
1. Target file: `Inbox/VideoNotes.md`.
2. Convert `28:53` -> `1733`s.
3. Convert `12:41` (761s) -> `761`s.
4. Output: "已转换笔记格式：\n- [28:53](obsidian://cloud-video-seek?file=Inbox%2FVideoNotes.md&ts=1733) 龙城之战与情报困境\n- [12:41](obsidian://cloud-video-seek?file=Inbox%2FVideoNotes.md&ts=761) 关键结论"

**User**: 把这段百度网盘的 AI 总结转一下，视频在 `Lectures/Physics.md`：
00:08 课程大纲介绍
05:42 经典力学三大定律
15:20 万有引力公式推导
**Process**:
1. Target file: `Lectures/Physics.md`.
2. Parse 00:08 (8s), 05:42 (342s), 15:20 (920s).
3. Output: "已转换 AI 总结：\n- [00:08](obsidian://cloud-video-seek?file=Lectures%2FPhysics.md&ts=8) 课程大纲介绍\n- [05:42](obsidian://cloud-video-seek?file=Lectures%2FPhysics.md&ts=342) 经典力学三大定律\n- [15:20](obsidian://cloud-video-seek?file=Lectures%2FPhysics.md&ts=920) 万有引力公式推导"

**User**: 搜索库里所有关于“神经网络”的视频标记。
**Process**:
1. Call `search_files(query: "obsidian://cloud-video-seek 神经网络")`.
2. Parse the results and group them by video.
3. Output: "在以下视频中找到了相关标记：\n- **DeepLearning.md**: [05:00] 神经网络基础..."

**User**: 我想在 Canvas 里加一个视频跳转，怎么写？
**Process**:
1. Explain that Canvas text nodes support the same `obsidian://cloud-video-seek` format.
2. Provide an example link: `obsidian://cloud-video-seek?file=CanvasFile.canvas&ts=120`.

**User**: 跳转到库中文件 `Movies/Inception.md` 的 1小时20分。
**Process**:
1. Convert `01:20:00` to `4800` seconds.
2. Provide the seek link: `obsidian://cloud-video-seek?file=Movies%2FInception.md&ts=4800`.