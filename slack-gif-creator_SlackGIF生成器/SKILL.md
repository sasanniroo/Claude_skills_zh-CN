---
name: slack-gif-creator
description: 面向 Slack 优化的动图制作工具包，内置尺寸校验器与可组合的动画基元。当用户提出“帮我做一个在 Slack 上展示 X 做 Y 的 GIF”之类的请求时，即应使用本技能生成 Slack 动图或表情动画。
license: 完整条款见 LICENSE.txt
---

# Slack GIF 生成器 - 灵活的工具箱

一个专为 Slack 优化的动图工具包，提供 Slack 约束校验器、可组合的动画基元，以及可选的辅助工具。**可根据创意任意组合这些工具，实现目标效果。**

## Slack 的动图要求

Slack 对不同用途的 GIF 有特定要求：

**消息动图：**
- 文件大小上限约 2MB
- 最佳尺寸：480×480
- 常见帧率：15-20 FPS
- 色彩限制：128-256
- 推荐时长：2-5 秒

**表情动图：**
- 文件大小上限 64KB（非常严格）
- 最佳尺寸：128×128
- 常见帧率：10-12 FPS
- 色彩限制：32-48
- 推荐时长：1-2 秒

**表情动图特别困难**——64KB 限制极其严格。建议策略：
- 总帧数控制在 10-15 帧
- 色彩数量不超过 32-48
- 设计尽量简洁
- 避免渐变
- 频繁检查文件大小

## 工具箱结构

本技能提供三类工具：

1. **校验器** —— 检查 GIF 是否符合 Slack 要求
2. **动画基元** —— 可组合的动效组件（抖动、弹跳、移动、万花筒等）
3. **辅助工具** —— 可选的通用函数（文字、配色、特效等）

**如何组合完全由创意决定。**

## 核心校验器

要确保 GIF 满足 Slack 限制，可使用以下校验器：

```python
from core.gif_builder import GIFBuilder

# 创建 GIF 后，验证是否合规
builder = GIFBuilder(width=128, height=128, fps=10)
# ... 以任意方式添加帧 ...

# 保存并检查大小
info = builder.save('emoji.gif', num_colors=48, optimize_for_emoji=True)

# save 方法会在超限时自动警告
# info 字典提供 size_kb、size_mb、frame_count、duration_seconds 等信息
```

**文件大小校验：**
```python
from core.validators import check_slack_size

# 判断 GIF 是否符合大小限制
passes, info = check_slack_size('emoji.gif', is_emoji=True)
# 返回 (True/False, 含大小信息的字典)
```

**尺寸校验：**
```python
from core.validators import validate_dimensions

# 检查宽高
passes, info = validate_dimensions(128, 128, is_emoji=True)
# 返回 (True/False, 含尺寸信息的字典)
```

**完整校验：**
```python
from core.validators import validate_gif, is_slack_ready

# 执行全部校验
all_pass, results = validate_gif('emoji.gif', is_emoji=True)

# 快速检查
if is_slack_ready('emoji.gif', is_emoji=True):
    print("Ready to upload!")
```

## 动画基元

以下基元可随意组合，作用于任何对象：

### Shake（抖动）
```python
from templates.shake import create_shake_animation

# 抖动表情
frames = create_shake_animation(
    object_type='emoji',
    object_data={'emoji': '😱', 'size': 80},
    num_frames=20,
    shake_intensity=15,
    direction='both'  # 可选 'horizontal'、'vertical'
)
```

### Bounce（弹跳）
```python
from templates.bounce import create_bounce_animation

# 弹跳圆形
frames = create_bounce_animation(
    object_type='circle',
    object_data={'radius': 40, 'color': (255, 100, 100)},
    num_frames=30,
    bounce_height=150
)
```

### Spin / Rotate（旋转）
```python
from templates.spin import create_spin_animation, create_loading_spinner

# 顺时针旋转
frames = create_spin_animation(
    object_type='emoji',
    object_data={'emoji': '🔄', 'size': 100},
    rotation_type='clockwise',
    full_rotations=2
)

# 摇摆式旋转
frames = create_spin_animation(rotation_type='wobble', full_rotations=3)

# 加载旋转器
frames = create_loading_spinner(spinner_type='dots')
```

### Pulse / Heartbeat（脉动 / 心跳）
```python
from templates.pulse import create_pulse_animation, create_attention_pulse

# 平滑脉动
frames = create_pulse_animation(
    object_data={'emoji': '❤️', 'size': 100},
    pulse_type='smooth',
    scale_range=(0.8, 1.2)
)

# 心跳（双跳节奏）
frames = create_pulse_animation(pulse_type='heartbeat')

# 表情动图专用的提醒脉动
frames = create_attention_pulse(emoji='⚠️', num_frames=20)
```

### Fade（淡入淡出）
```python
from templates.fade import create_fade_animation, create_crossfade

# 淡入
frames = create_fade_animation(fade_type='in')

# 淡出
frames = create_fade_animation(fade_type='out')

# 两个表情的交叉淡化
frames = create_crossfade(
    object1_data={'emoji': '😊', 'size': 100},
    object2_data={'emoji': '😂', 'size': 100}
)
```

### Zoom（缩放）
```python
from templates.zoom import create_zoom_animation, create_explosion_zoom

# 强烈放大
frames = create_zoom_animation(
    zoom_type='in',
    scale_range=(0.1, 2.0),
    add_motion_blur=True
)

# 缩小
frames = create_zoom_animation(zoom_type='out')

# 爆炸式缩放
frames = create_explosion_zoom(emoji='💥')
```

### Explode / Shatter（爆裂 / 破碎）
```python
from templates.explode import create_explode_animation, create_particle_burst

# 爆裂
frames = create_explode_animation(
    explode_type='burst',
    num_pieces=25
)

# 破碎效果
frames = create_explode_animation(explode_type='shatter')

# 化为粒子
frames = create_explode_animation(explode_type='dissolve')

# 粒子爆发
frames = create_particle_burst(particle_count=30)
```

### Wiggle / Jiggle（摆动 / 抖颤）
```python
from templates.wiggle import create_wiggle_animation, create_excited_wiggle

# 果冻颤动
frames = create_wiggle_animation(
    wiggle_type='jello',
    intensity=1.0,
    cycles=2
)

# 波浪运动
frames = create_wiggle_animation(wiggle_type='wave')

# 表情动图的兴奋摇摆
frames = create_excited_wiggle(emoji='🎉')
```

### Slide（滑入 / 滑出）
```python
from templates.slide import create_slide_animation, create_multi_slide

# 左侧滑入并带回弹
frames = create_slide_animation(
    direction='left',
    slide_type='in',
    overshoot=True
)

# 横向滑动
frames = create_slide_animation(direction='left', slide_type='across')

# 多个对象依次滑入
objects = [
    {'data': {'emoji': '🎯', 'size': 60}, 'direction': 'left', 'final_pos': (120, 240)},
    {'data': {'emoji': '🎪', 'size': 60}, 'direction': 'right', 'final_pos': (240, 240)}
]
frames = create_multi_slide(objects, stagger_delay=5)
```

### Flip（翻转）
```python
from templates.flip import create_flip_animation, create_quick_flip

# 两个表情的水平翻转
frames = create_flip_animation(
    object1_data={'emoji': '😊', 'size': 120},
    object2_data={'emoji': '😂', 'size': 120},
    flip_axis='horizontal'
)

# 垂直翻转
frames = create_flip_animation(flip_axis='vertical')

# 表情动图快速翻转
frames = create_quick_flip('👍', '👎')
```

### Morph / Transform（形变 / 转场）
```python
from templates.morph import create_morph_animation, create_reaction_morph

# 交叉淡化形变
frames = create_morph_animation(
    object1_data={'emoji': '😊', 'size': 100},
    object2_data={'emoji': '😂', 'size': 100},
    morph_type='crossfade'
)

# 缩放形变（一个缩小、另一个放大）
frames = create_morph_animation(morph_type='scale')

# 旋转形变（类似 3D 翻转）
frames = create_morph_animation(morph_type='spin_morph')
```

### Move Effect（运动轨迹）
```python
from templates.move import create_move_animation

# 线性移动
frames = create_move_animation(
    object_type='emoji',
    object_data={'emoji': '🚀', 'size': 60},
    start_pos=(50, 240),
    end_pos=(430, 240),
    motion_type='linear',
    easing='ease_out'
)

# 抛物线运动
frames = create_move_animation(
    object_type='emoji',
    object_data={'emoji': '⚽', 'size': 60},
    start_pos=(50, 350),
    end_pos=(430, 350),
    motion_type='arc',
    motion_params={'arc_height': 150}
)

# 圆周运动
frames = create_move_animation(
    object_type='emoji',
    object_data={'emoji': '🌍', 'size': 50},
    motion_type='circle',
    motion_params={
        'center': (240, 240),
        'radius': 120,
        'angle_range': 360  # 完整一圈
    }
)

# 波浪运动
frames = create_move_animation(
    motion_type='wave',
    motion_params={
        'wave_amplitude': 50,
        'wave_frequency': 2
    }
)

# 也可以使用底层缓动函数
from core.easing import interpolate, calculate_arc_motion

for i in range(num_frames):
    t = i / (num_frames - 1)
    x = interpolate(start_x, end_x, t, easing='ease_out')
    # 或者：x, y = calculate_arc_motion(start, end, height, t)
```

### Kaleidoscope（万花筒效果）
```python
from templates.kaleidoscope import apply_kaleidoscope, create_kaleidoscope_animation

# 应用于单帧
kaleido_frame = apply_kaleidoscope(frame, segments=8)

# 生成万花筒动画
frames = create_kaleidoscope_animation(
    base_frame=my_frame,  # 或 None 使用示例图案
    num_frames=30,
    segments=8,
    rotation_speed=1.0
)

# 简易镜像效果（性能更高）
from templates.kaleidoscope import apply_simple_mirror

mirrored = apply_simple_mirror(frame, mode='quad')  # 四向镜像
# mode 可选：'horizontal'、'vertical'、'quad'、'radial'
```

**自由组合基元可参考以下模式：**
```python
# 示例：弹跳 + 抖动，营造冲击力
for i in range(num_frames):
    frame = create_blank_frame(480, 480, bg_color)

    # 弹跳运动
    t_bounce = i / (num_frames - 1)
    y = interpolate(start_y, ground_y, t_bounce, 'bounce_out')

    # 落地瞬间增加抖动
    if y >= ground_y - 5:
        shake_x = math.sin(i * 2) * 10
        x = center_x + shake_x
    else:
        x = center_x

    draw_emoji(frame, '⚽', (x, y), size=60)
    builder.add_frame(frame)
```

## 辅助工具

以下工具可选用、改写或替换，视需求而定。

### GIF Builder（组装与优化）

```python
from core.gif_builder import GIFBuilder

# 根据需求创建 builder
builder = GIFBuilder(width=480, height=480, fps=20)

# 添加帧
for frame in my_frames:
    builder.add_frame(frame)

# 保存并优化
builder.save('output.gif',
             num_colors=128,
             optimize_for_emoji=False)
```

主要特性：
- 自动色彩量化
- 删除重复帧
- 超限时发出警告
- Emoji 模式（更激进的压缩）

### 文字渲染

对于小尺寸（如表情动图），文字可读性很难保证。常见做法是加描边：

```python
from core.typography import draw_text_with_outline, TYPOGRAPHY_SCALE

# 带描边的文字
draw_text_with_outline(
    frame, "BONK!",
    position=(240, 100),
    font_size=TYPOGRAPHY_SCALE['h1'],  # 60px
    text_color=(255, 68, 68),
    outline_color=(0, 0, 0),
    outline_width=4,
    centered=True
)
```

如需自定义文字，可直接使用 PIL 的 `ImageDraw.text()`，适用于较大动图。

### 色彩管理

专业动图通常使用和谐的调色板：

```python
from core.color_palettes import get_palette

# 获取预制色板
palette = get_palette('vibrant')  # 还可选 'pastel'、'dark'、'neon'、'professional'

bg_color = palette['background']
text_color = palette['primary']
accent_color = palette['accent']
```

也可直接使用 RGB 元组，按需求取用。

### 视觉特效

用于强调时刻的可选特效：

```python
from core.visual_effects import ParticleSystem, create_impact_flash, create_shockwave_rings

# 粒子系统
particles = ParticleSystem()
particles.emit_sparkles(x=240, y=200, count=15)
particles.emit_confetti(x=240, y=200, count=20)

# 每帧更新渲染
particles.update()
particles.render(frame)

# 闪光效果
frame = create_impact_flash(frame, position=(240, 200), radius=100)

# 冲击波环
frame = create_shockwave_rings(frame, position=(240, 200), radii=[30, 60, 90])
```

### 缓动函数

为了获得流畅运动，应使用缓动而非线性插值：

```python
from core.easing import interpolate

# 下落（加速）
y = interpolate(start=0, end=400, t=progress, easing='ease_in')

# 落地（减速）
y = interpolate(start=0, end=400, t=progress, easing='ease_out')

# 弹跳
y = interpolate(start=0, end=400, t=progress, easing='bounce_out')

# 过冲弹性
scale = interpolate(start=0.5, end=1.0, t=progress, easing='elastic_out')
```

可用缓动包括：`linear`、`ease_in`、`ease_out`、`ease_in_out`、`bounce_out`、`elastic_out`、`back_out`（过冲）等，详见 `core/easing.py`。

### 帧合成

如需基础绘制工具，可使用：

```python
from core.frame_composer import (
    create_gradient_background,  # 渐变背景
    draw_emoji_enhanced,         # 搭配阴影的表情
    draw_circle_with_shadow,     # 具备深度的图形
    draw_star                    # 五角星
)

# 渐变背景
frame = create_gradient_background(480, 480, top_color, bottom_color)

# 带阴影的表情
draw_emoji_enhanced(frame, '🎉', position=(200, 200), size=80, shadow=True)
```

## 优化策略

若 GIF 过大，可尝试：

**消息动图（>2MB）**
1. 减少帧数（降低 FPS 或缩短时长）
2. 减少色彩（从 128 降到 64）
3. 调整尺寸（480×480 → 320×320）
4. 启用重复帧去重

**表情动图（>64KB）——必须激进压缩：**
1. 控制在 10-12 帧
2. 色彩不超过 32-40
3. 避免渐变（纯色更易压缩）
4. 简化设计（更少元素）
5. 调用 `save` 时设置 `optimize_for_emoji=True`

## 组合示例

### 简单反应（脉动）
```python
builder = GIFBuilder(128, 128, 10)

for i in range(12):
    frame = Image.new('RGB', (128, 128), (240, 248, 255))

    # 脉动缩放
    scale = 1.0 + math.sin(i * 0.5) * 0.15
    size = int(60 * scale)

    draw_emoji_enhanced(frame, '😱', position=(64-size//2, 64-size//2),
                       size=size, shadow=False)
    builder.add_frame(frame)

builder.save('reaction.gif', num_colors=40, optimize_for_emoji=True)

# 校验
from core.validators import check_slack_size
check_slack_size('reaction.gif', is_emoji=True)
```

### 动作与冲击（弹跳 + 闪光）
```python
builder = GIFBuilder(480, 480, 20)

# 阶段 1：物体下落
for i in range(15):
    frame = create_gradient_background(480, 480, (240, 248, 255), (200, 230, 255))
    t = i / 14
    y = interpolate(0, 350, t, 'ease_in')
    draw_emoji_enhanced(frame, '⚽', position=(220, int(y)), size=80)
    builder.add_frame(frame)

# 阶段 2：撞击与闪光
for i in range(8):
    frame = create_gradient_background(480, 480, (240, 248, 255), (200, 230, 255))

    # 前几帧添加闪光
    if i < 3:
        frame = create_impact_flash(frame, (240, 350), radius=120, intensity=0.6)

    draw_emoji_enhanced(frame, '⚽', position=(220, 350), size=80)

    # 显示文字
    if i > 2:
        draw_text_with_outline(frame, "GOAL!", position=(240, 150),
                              font_size=60, text_color=(255, 68, 68),
                              outline_color=(0, 0, 0), outline_width=4, centered=True)

    builder.add_frame(frame)

builder.save('goal.gif', num_colors=128)
```

### 组合基元（移动 + 抖动）
```python
from templates.shake import create_shake_animation

# 创建抖动动画
shake_frames = create_shake_animation(
    object_type='emoji',
    object_data={'emoji': '😰', 'size': 70},
    num_frames=20,
    shake_intensity=12
)

# 创建触发抖动的移动元素
builder = GIFBuilder(480, 480, 20)
for i in range(40):
    t = i / 39

    if i < 20:
        # 触发前：空背景 + 移动对象
        frame = create_blank_frame(480, 480, (255, 255, 255))
        x = interpolate(50, 300, t * 2, 'linear')
        draw_emoji_enhanced(frame, '🚗', position=(int(x), 300), size=60)
        draw_emoji_enhanced(frame, '😰', position=(350, 200), size=70)
    else:
        # 触发后：使用抖动帧
        frame = shake_frames[i - 20]
        # 将汽车固定在终点
        draw_emoji_enhanced(frame, '🚗', position=(300, 300), size=60)

    builder.add_frame(frame)

builder.save('scare.gif')
```

## 制作理念

本工具箱提供的是“组件”，而非固定模板。制作 GIF 时：

1. **理解创意目标** —— 想表现什么？整体氛围如何？
2. **设计动画结构** —— 分解为预备、动作、反应等阶段
3. **按需组合基元** —— 抖动、弹跳、移动、特效任意叠加
4. **验证限制** —— 特别注意表情动图的大小
5. **必要时迭代** —— 超出限制时减少帧数 / 色彩

**目标是在 Slack 的技术约束内，保持创作自由。**

## 依赖项

如需使用本工具箱，请确保安装以下依赖（若已安装可跳过）：

```bash
pip install pillow imageio numpy
```
