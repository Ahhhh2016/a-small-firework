### A Small Firework (English)

A small Three.js + WebAudio + GLSL experiment (inspired by Shadertoy) featuring four aerial fireworks (One Spark / Bloom / Drift / Scatter) and a handheld Sparkler, paired with lightweight synthesized sound effects.


### How to Run

- Simply open `index.html` in your browser.
- Your first click/interaction enables audio (required by browser policies).


### Controls

- Left panel buttons:
  - One Spark (single-point burst)
  - Bloom (uniform bloom)
  - Drift (planar, drifting bloom)
  - Scatter (explosive scatter)
  - Sparkler (handheld, continuous burn)
- Click the canvas to launch the selected firework. You can toggle Auto Play, and adjust volume (top-right).


### Performance

- Default pixel ratio is capped to reduce GPU load on high-DPI screens:
  - `renderer.setPixelRatio(Math.min(window.devicePixelRatio, 1))`
- While Sparkler is active, pixel ratio is temporarily lowered to `0.75` and restored on stop.


### Physics and Rendering Highlights (with brief formulas)

- Core particle update
  - Discrete Euler integration with gravity and air drag:
    - \( v_y \mathrel{+}= g \)
    - \( \mathbf{v} \mathrel{*}= \text{drag} \) (per-frame damping factor < 1)
    - \( \mathbf{x} \mathrel{+}= \mathbf{v} \)
  - Continuous approximation (for intuition): \( \mathbf{x}(t) \approx \mathbf{x}_0 + \mathbf{v}_0 t e^{-k t} + \tfrac{1}{2}\mathbf{g} t^2 \).

- One Spark (type: `spark`)
  - Isotropic initial velocity (uniform on a sphere):
    - \( \theta \sim U[0,2\pi),\ \phi = \arccos(2u-1) \), \( \mathbf{v}_0 = r[\sin\phi\cos\theta,\ \sin\phi\sin\theta,\ \cos\phi] \).
  - Medium gravity and damping; mid/late life spawns trail points to emphasize streaks.

- Bloom (type: `bloom`)
  - Fibonacci sphere distribution for near-uniform directions:
    - \( y = 1 - \frac{i}{N-1}\cdot 2,\ r=\sqrt{1-y^2},\ \theta = \varphi i \) (golden angle \(\varphi\)),
      \( \mathbf{v}_0 = \text{base} \cdot (r\cos\theta,\ y,\ r\sin\theta) \).
  - Lower gravity and gentle damping; particle size eases in over the first ~60 frames (cubic ease).

- Drift (type: `drift`)
  - Near-ring distribution with very small vertical component:
    - \( \theta \sim U[0,2\pi),\ \mathbf{v}_0 \approx (r\cos\theta,\ \epsilon,\ r\sin\theta) \) → planar drift.
  - Very small gravity, slightly stronger damping; an overall “lighter” drifting feel.

- Scatter (type: `scatter`)
  - Random direction on the unit sphere with larger magnitude → more explosive scatter.
  - Default gravity, medium-to-strong damping; paired with more aggressive crackle SFX.

- Sparkler (handheld, shader-driven)
  - Fragment/vertex shaders blend screen-space glow streaks and noise-based detail:
    - Burn line descends over time: \( y_\text{burn}(t) = y_0 - s t \);
    - Emission transitions from a narrow ring-cone to isotropic;
    - Damped ballistic path: \( \mathbf{x}(t) \approx \mathbf{x}_0 + \mathbf{v}_0 t e^{-k t} + \tfrac{1}{2}\mathbf{g}t^2 \);
    - Occasional sub-spark “micro-bursts” and splatter, noise-driven flicker intensity;
    - Paper/smoke grain via 3D noise, triplanar mapping, and time-animated noise.


### Sound Design (Web Audio)

- Bus and volume: `AudioContext` + `masterGain`. Audio is initialized on first interaction via `initAudio()`.
- Launch/Ascent (general):
  - `playSoftLaunch()`: white noise → high-pass filter (swept up) → short exponential amplitude envelope; mimics nozzle/jet wind.

- Explosion flavors:
  - Bloom: `playSoftBloom()` multiple short sine partials layered for a soft, rounded “bloom” timbre.
  - Drift: `playShimmer()` triangle wave + slower release envelope for a subtle shimmering quality.
  - Scatter: `playSoftCrackle()` repeated short square-wave blips for continuous crackle.
  - One Spark: a denser burst of `playSoftCrackle()` calls to highlight fine, punctate pops.
  - Sparkler:
    - `startSparklerBurn()`: looped white noise → band-pass + low gain for a continuous burn bed;
    - `sparklerCrackle()`: random one-shot white noise → high-pass + fast envelope for micro-pops.


### Stack

- Three.js (particle points, additive blending, trail ring buffer).
- Web Audio API (noise sources, oscillators, filters, exponential envelopes).
- GLSL shaders (noise-driven spark/smoke glow, sub-jet details), Shadertoy-inspired.


### References

- Shadertoy works that inspired shader structure/noise/glow:
  - `https://www.shadertoy.com/view/csdfR8`
  - `https://www.shadertoy.com/view/wdlGW4`
  - `https://www.shadertoy.com/view/tsX3DN`
  - `https://www.shadertoy.com/view/lsd3RX`

(All audio is synthesized in real time; no external samples.)

### A Small Firework

一个用 Three.js + WebAudio + GLSL（受 Shadertoy 启发）的烟花/手持烟花小实验。包含 4 种烟花弹（One Spark / Bloom / Drift / Scatter）与 1 种手持烟花（Sparkler）表现，并配套了简洁的合成音效。


### 运行方式

- 直接用浏览器打开 `index.html` 即可。
- 第一次点击页面或按钮会激活音频（浏览器策略要求用户手势）。


### 交互说明

- 左侧按钮：
  - One Spark（单点爆）
  - Bloom（均匀绽放）
  - Drift（漂移绽放）
  - Scatter（散射爆裂）
  - Sparkler（手持烟花，持续燃烧效果）
- 点击画面发射选中的烟花；也可开启/暂停自动播放；右上角可调音量。


### 性能

- 为降低高 DPI 屏上的 GPU 压力，默认像素比进行了封顶：
  - `renderer.setPixelRatio(Math.min(window.devicePixelRatio, 1))`
- Sparkler 运行期间会临时把像素比降到 `0.75`，结束后恢复原像素比。


### 物理与渲染要点（含简要公式）

- 基础运动（粒子更新）
  - 使用离散欧拉积分，包含重力与空气阻力：
    - \( v_y \mathrel{+}= g \)
    - \( \mathbf{v} \mathrel{*}= \text{drag} \)（每帧乘以 < 1 的阻尼系数）
    - \( \mathbf{x} \mathrel{+}= \mathbf{v} \)
  - 连续近似（仅说明）：\( \mathbf{x}(t) \approx \mathbf{x}_0 + \mathbf{v}_0 t e^{-k t} + \tfrac{1}{2}\mathbf{g} t^2 \)。

- One Spark（type: `spark`）
  - 初速度各向同性采样（球面均匀）：
    - \( \theta \sim U[0,2\pi),\ \phi = \arccos(2u-1) \)，\( \mathbf{v}_0 = r[\sin\phi\cos\theta,\ \sin\phi\sin\theta,\ \cos\phi] \)。
  - 物理参数：中等重力、中等阻尼；在寿命中后期添加轨迹粒子，强调拖尾。

- Bloom（type: `bloom`）
  - 使用 Fibonacci Sphere 近似均匀分布：
    - \( y = 1 - \frac{i}{N-1}\cdot 2,\ r=\sqrt{1-y^2},\ \theta = \varphi i \)（\(\varphi\) 为黄金角），
      \( \mathbf{v}_0 = \text{base} \cdot (r\cos\theta,\ y,\ r\sin\theta) \)。
  - 物理参数：较小重力、较弱阻尼；粒子大小在前 60 帧缓入至目标尺寸（立方缓动）。

- Drift（type: `drift`）
  - 初速度近似“环形”分布，垂直分量很小：
    - \( \theta \sim U[0,2\pi),\ \mathbf{v}_0 \approx (r\cos\theta,\ \epsilon,\ r\sin\theta) \)，形成平面漂移感。
  - 物理参数：重力很小、阻尼略强；整体更“轻”的漂移效果。

- Scatter（type: `scatter`）
  - 初速度随机方向（单位球面均匀）并放大幅度，形成爆裂散射。
  - 物理参数：重力默认、中到强阻尼；并伴随更明显的爆裂声（见下）。

- Sparkler（手持烟花，Shader）
  - 使用片元/顶点着色器进行屏幕空间光迹混合与体感噪声，核心包含：
    - 燃点随时间沿 \( y \) 轴向下移动：\( y_\text{burn}(t) = y_0 - s t \)；
    - 发射方向从“窄环锥混入各向同性”过渡；
    - 轨迹采用带指数阻尼的弹道近似：\( \mathbf{x}(t) \approx \mathbf{x}_0 + \mathbf{v}_0 t e^{-k t} + \tfrac{1}{2}\mathbf{g}t^2 \)；
    - 次级“微喷/微爆”与“溅射”片段，使用噪声驱动强度与闪烁；
    - 纸张/烟雾颗粒感通过 3D 噪声、三平面贴图与时间噪声抖动塑造。


### 声音设计（Web Audio）

- 总线与音量：`AudioContext` + `masterGain`。首次交互调用 `initAudio()`。
- 发射/上升（通用）：
  - `playSoftLaunch()`：白噪声 → 高通滤波（频率上扫）→ 短包络增益，模拟喷口风噪与推进。
- 不同爆炸类型：
  - Bloom：`playSoftBloom()` 多个正弦短音叠加，温和的“绽放”层次。
  - Drift：`playShimmer()` 三角波 + 慢释包络，带轻微“闪耀/颤音”感。
  - Scatter：`playSoftCrackle()` 多次触发的方波短音，形成连续“噼啪”爆裂。
  - One Spark：更密集的 `playSoftCrackle()` 阵列，突出细碎的爆点。
  - Sparkler：
    - `startSparklerBurn()`：循环白噪声 → 带通滤波 + 低电平增益，持续燃烧底噪；
    - `sparklerCrackle()`：随机单发白噪声 → 高通滤波 + 快速包络，模拟微型爆点。


### 技术栈

- Three.js（粒子系统、点精灵、加法混合、轨迹缓冲环）。
- Web Audio API（噪声源、振荡器、滤波器、指数包络）。
- GLSL Shader（火花/烟雾的噪声、光晕、次级喷射等，受 Shadertoy 启发）。


### 参考与致谢

- 受以下 Shadertoy 作品启发（着色思路/噪声/光迹等）：
  - `https://www.shadertoy.com/view/csdfR8`
  - `https://www.shadertoy.com/view/wdlGW4`
  - `https://www.shadertoy.com/view/tsX3DN`
  - `https://www.shadertoy.com/view/lsd3RX`

（本项目所有音效均为实时合成，无外部音频素材。）


