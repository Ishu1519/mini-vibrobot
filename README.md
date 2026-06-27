[index (1).html](https://github.com/user-attachments/files/29420889/index.1.html)
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mini VibroBot — Single-Motor BLE Micro Robot</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/3.2.2/es5/tex-mml-chtml.min.js"></script>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap');

  :root {
    --bg: #080810;
    --surface: #0f0f1a;
    --surface2: #151525;
    --cyan: #00e5ff;
    --orange: #ff6b35;
    --white: #eaeaea;
    --muted: #555570;
    --border: #1e1e35;
    --glow: 0 0 30px rgba(0,229,255,0.15);
  }

  * { margin: 0; padding: 0; box-sizing: border-box; }

  html { scroll-behavior: smooth; }

  body {
    background: var(--bg);
    color: var(--white);
    font-family: 'Space Grotesk', sans-serif;
    overflow-x: hidden;
    line-height: 1.7;
  }

  /* ── NAV ── */
  nav {
    position: fixed; top: 0; width: 100%; z-index: 100;
    padding: 1rem 2rem;
    display: flex; justify-content: space-between; align-items: center;
    background: rgba(8,8,16,0.85);
    backdrop-filter: blur(12px);
    border-bottom: 1px solid var(--border);
  }
  .nav-logo {
    font-size: 1rem; font-weight: 600; color: var(--cyan);
    letter-spacing: 0.05em;
  }
  .nav-links { display: flex; gap: 2rem; list-style: none; }
  .nav-links a {
    color: var(--muted); text-decoration: none; font-size: 0.85rem;
    font-weight: 500; letter-spacing: 0.03em; transition: color 0.2s;
  }
  .nav-links a:hover { color: var(--white); }

  /* ── HERO ── */
  .hero {
    min-height: 100vh;
    display: flex; flex-direction: column;
    justify-content: center; align-items: center;
    text-align: center; padding: 6rem 2rem 4rem;
    position: relative; overflow: hidden;
  }

  .hero-grid {
    position: absolute; inset: 0;
    background-image:
      linear-gradient(rgba(0,229,255,0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,229,255,0.03) 1px, transparent 1px);
    background-size: 40px 40px;
    animation: gridDrift 20s linear infinite;
  }

  @keyframes gridDrift {
    from { transform: translateY(0); }
    to   { transform: translateY(40px); }
  }

  .hero-eyebrow {
    font-size: 0.75rem; font-weight: 500; letter-spacing: 0.15em;
    color: var(--cyan); text-transform: uppercase;
    margin-bottom: 1.5rem; opacity: 0;
    animation: fadeUp 0.8s 0.2s forwards;
  }

  .hero-title {
    font-size: clamp(2.5rem, 7vw, 5rem);
    font-weight: 700; line-height: 1.05;
    letter-spacing: -0.02em;
    max-width: 800px;
    opacity: 0; animation: fadeUp 0.8s 0.4s forwards;
  }

  .hero-title span { color: var(--cyan); }

  .hero-sub {
    margin-top: 1.5rem;
    font-size: 1.1rem; color: var(--muted); max-width: 560px;
    font-weight: 400;
    opacity: 0; animation: fadeUp 0.8s 0.6s forwards;
  }

  .hero-badges {
    display: flex; gap: 0.75rem; flex-wrap: wrap;
    justify-content: center; margin-top: 2.5rem;
    opacity: 0; animation: fadeUp 0.8s 0.8s forwards;
  }

  .badge {
    padding: 0.35rem 0.9rem; border-radius: 999px;
    font-size: 0.75rem; font-weight: 600; letter-spacing: 0.05em;
    font-family: 'JetBrains Mono', monospace;
  }
  .badge-cyan  { background: rgba(0,229,255,0.1); color: var(--cyan); border: 1px solid rgba(0,229,255,0.3); }
  .badge-orange{ background: rgba(255,107,53,0.1); color: var(--orange); border: 1px solid rgba(255,107,53,0.3); }
  .badge-white { background: rgba(255,255,255,0.05); color: var(--white); border: 1px solid var(--border); }

  /* ── ROBOT ANIMATION ── */
  .robot-viz {
    margin-top: 4rem; position: relative;
    opacity: 0; animation: fadeUp 0.8s 1s forwards;
  }

  .osc-canvas {
    width: 100%; max-width: 700px; height: 120px;
    display: block; margin: 0 auto;
  }

  /* ── SECTIONS ── */
  section {
    max-width: 900px; margin: 0 auto;
    padding: 6rem 2rem;
  }

  .section-eyebrow {
    font-size: 0.7rem; font-weight: 600; letter-spacing: 0.2em;
    color: var(--cyan); text-transform: uppercase; margin-bottom: 0.75rem;
  }

  h2 {
    font-size: clamp(1.8rem, 4vw, 2.5rem);
    font-weight: 700; letter-spacing: -0.02em;
    margin-bottom: 1.5rem; line-height: 1.1;
  }

  h3 {
    font-size: 1.1rem; font-weight: 600; margin: 2rem 0 0.75rem;
    color: var(--white);
  }

  p { color: #a0a0b8; margin-bottom: 1rem; }

  /* ── STICK-SLIP DIAGRAM ── */
  .diagram-wrap {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 12px; padding: 2rem;
    margin: 2.5rem 0;
    position: relative; overflow: hidden;
  }

  .diagram-wrap::before {
    content: '';
    position: absolute; inset: 0;
    background: radial-gradient(ellipse at 50% 0%, rgba(0,229,255,0.05) 0%, transparent 70%);
    pointer-events: none;
  }

  .robot-svg { width: 100%; max-width: 600px; display: block; margin: 0 auto; }

  /* ── CONTROL TABLE ── */
  .control-grid {
    display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
    gap: 1rem; margin: 2rem 0;
  }

  .control-card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 10px; padding: 1.25rem;
    text-align: center; transition: border-color 0.2s, transform 0.2s;
  }
  .control-card:hover { border-color: var(--cyan); transform: translateY(-2px); }

  .control-icon { font-size: 2rem; margin-bottom: 0.5rem; }
  .control-label { font-size: 0.75rem; color: var(--muted); letter-spacing: 0.1em; text-transform: uppercase; }
  .control-cmd {
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.85rem; color: var(--cyan); margin-top: 0.5rem;
  }

  /* ── MATH SECTION ── */
  .math-section { background: var(--surface); border-left: 3px solid var(--cyan); border-radius: 0 8px 8px 0; padding: 1.5rem 2rem; margin: 1.5rem 0; }
  .math-section .label { font-size: 0.7rem; color: var(--cyan); letter-spacing: 0.1em; text-transform: uppercase; margin-bottom: 0.5rem; font-weight: 600; }
  .math-section p { color: var(--muted); font-size: 0.9rem; }

  .eq-box {
    background: var(--surface2);
    border: 1px solid var(--border);
    border-radius: 8px; padding: 1.25rem 1.5rem; margin: 1rem 0;
    font-size: 1.1rem; text-align: center; overflow-x: auto;
  }

  /* ── HARDWARE TABLE ── */
  .hw-table { width: 100%; border-collapse: collapse; margin: 1.5rem 0; }
  .hw-table th { text-align: left; padding: 0.75rem 1rem; font-size: 0.7rem; letter-spacing: 0.1em; text-transform: uppercase; color: var(--muted); border-bottom: 1px solid var(--border); }
  .hw-table td { padding: 0.85rem 1rem; border-bottom: 1px solid var(--border); font-size: 0.9rem; color: #c0c0d8; }
  .hw-table tr:hover td { background: var(--surface); }
  .hw-table code { font-family: 'JetBrains Mono', monospace; font-size: 0.8rem; color: var(--cyan); background: rgba(0,229,255,0.08); padding: 0.15rem 0.4rem; border-radius: 3px; }

  /* ── STATUS ── */
  .status-list { list-style: none; margin: 1.5rem 0; }
  .status-list li {
    display: flex; align-items: center; gap: 0.75rem;
    padding: 0.6rem 0; border-bottom: 1px solid var(--border);
    font-size: 0.9rem; color: #c0c0d8;
  }
  .status-list li:last-child { border-bottom: none; }
  .dot { width: 8px; height: 8px; border-radius: 50%; flex-shrink: 0; }
  .dot.done   { background: #22c55e; box-shadow: 0 0 8px #22c55e; }
  .dot.pending{ background: var(--muted); }
  .dot.active { background: var(--orange); box-shadow: 0 0 8px var(--orange); animation: pulse 1.5s infinite; }
  @keyframes pulse { 0%,100%{ opacity:1; } 50%{ opacity:0.4; } }

  /* ── CODE ── */
  pre {
    background: var(--surface2);
    border: 1px solid var(--border);
    border-radius: 8px; padding: 1.25rem 1.5rem;
    overflow-x: auto; margin: 1rem 0;
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.82rem; line-height: 1.7; color: #c0c0d8;
  }
  .kw { color: var(--cyan); }
  .fn { color: #a78bfa; }
  .cm { color: var(--muted); }
  .str{ color: #86efac; }

  /* ── DIVIDER ── */
  .divider {
    width: 100%; height: 1px;
    background: linear-gradient(90deg, transparent, var(--border), transparent);
    margin: 0;
  }

  /* ── FOOTER ── */
  footer {
    text-align: center; padding: 3rem 2rem;
    color: var(--muted); font-size: 0.85rem;
    border-top: 1px solid var(--border);
  }
  footer a { color: var(--cyan); text-decoration: none; }

  @keyframes fadeUp {
    from { opacity: 0; transform: translateY(20px); }
    to   { opacity: 1; transform: translateY(0); }
  }

  /* Scroll reveal */
  .reveal { opacity: 0; transform: translateY(30px); transition: opacity 0.7s, transform 0.7s; }
  .reveal.visible { opacity: 1; transform: none; }
</style>
</head>
<body>

<nav>
  <span class="nav-logo">⬡ Mini VibroBot</span>
  <ul class="nav-links">
    <li><a href="#how">How It Works</a></li>
    <li><a href="#math">Physics</a></li>
    <li><a href="#hardware">Hardware</a></li>
    <li><a href="#build">Build</a></li>
    <li><a href="https://github.com/Ishu1519" target="_blank">GitHub →</a></li>
  </ul>
</nav>

<!-- HERO -->
<div class="hero">
  <div class="hero-grid"></div>
  <p class="hero-eyebrow">Open Source Micro Robotics</p>
  <h1 class="hero-title">One motor.<br><span>Any direction.</span></h1>
  <p class="hero-sub">A BLE-controlled vibrobot smaller than a coin — steered entirely by reversing a single coin ERM motor mounted at a tilt. No wheels, no servos, no gears.</p>
  <div class="hero-badges">
    <span class="badge badge-cyan">BL602 · BLE 5.0</span>
    <span class="badge badge-orange">Coin ERM 10mm</span>
    <span class="badge badge-white">~20mm × 20mm</span>
    <span class="badge badge-white">80mAh LiPo</span>
    <span class="badge badge-cyan">Stick-Slip Locomotion</span>
  </div>

  <div class="robot-viz">
    <canvas class="osc-canvas" id="oscCanvas" width="700" height="120"></canvas>
  </div>
</div>

<div class="divider"></div>

<!-- HOW IT WORKS -->
<section id="how">
  <div class="reveal">
    <p class="section-eyebrow">Locomotion Principle</p>
    <h2>How one motor steers a robot</h2>
    <p>The eccentric mass in a coin ERM motor creates a rotating centrifugal force. By default, a coin motor vibrates mostly in the Z-axis (up-down). Mount it at a <strong>20-30° tilt</strong> above the chassis and that force gains a horizontal component — enough to overcome friction and slide the chassis forward. This is <strong>stick-slip locomotion</strong>, and the tilt angle is everything.</p>

    <div style="background:rgba(255,107,53,0.08); border:1px solid rgba(255,107,53,0.3); border-radius:8px; padding:1rem 1.25rem; margin:1.5rem 0; font-size:0.88rem; color:#ff6b35;">
      ⚠️ <strong>Critical:</strong> Coin ERM mounted flat = only vertical vibration = no steering. The 20-30° tilt is the entire physics basis. Never skip this.
    </div>

    <div class="diagram-wrap">
      <svg class="robot-svg" viewBox="0 0 600 220" fill="none" xmlns="http://www.w3.org/2000/svg">
        <!-- Ground -->
        <line x1="20" y1="170" x2="580" y2="170" stroke="#1e1e35" stroke-width="2"/>
        <text x="300" y="188" text-anchor="middle" fill="#555570" font-size="11" font-family="Space Grotesk">Surface (rigid, μs > μk)</text>

        <!-- Chassis -->
        <rect x="200" y="130" width="200" height="40" rx="4" fill="#0f0f1a" stroke="#1e1e35" stroke-width="1.5"/>
        <text x="300" y="154" text-anchor="middle" fill="#555570" font-size="10" font-family="Space Grotesk">BL602 Chassis (20mm × 20mm)</text>

        <!-- Pen nibs -->
        <circle cx="225" cy="170" r="4" fill="#00e5ff" opacity="0.6"/>
        <circle cx="255" cy="170" r="4" fill="#00e5ff" opacity="0.6"/>
        <circle cx="300" cy="170" r="4" fill="#00e5ff" opacity="0.6"/>
        <circle cx="345" cy="170" r="4" fill="#00e5ff" opacity="0.6"/>
        <circle cx="375" cy="170" r="4" fill="#00e5ff" opacity="0.6"/>

        <!-- Coin motor tilted (flat disc shape) -->
        <g transform="translate(285,108) rotate(-25)">
          <ellipse cx="0" cy="0" rx="18" ry="5" fill="#ff6b35" opacity="0.9"/>
          <ellipse cx="0" cy="-3" rx="18" ry="5" fill="#ff8050" opacity="0.9"/>
          <circle cx="8" cy="-3" r="4" fill="#ffaa80" id="eccMass"/>
        </g>
        <text x="195" y="100" fill="#ff6b35" font-size="10" font-family="Space Grotesk">Coin ERM (25° tilt)</text>
        <!-- Tilt angle indicator -->
        <line x1="270" y1="130" x2="310" y2="108" stroke="#ff6b35" stroke-width="1" stroke-dasharray="3,3" opacity="0.5"/>

        <!-- Force arrows -->
        <defs>
          <marker id="arrowC" markerWidth="6" markerHeight="6" refX="3" refY="3" orient="auto">
            <path d="M0,0 L6,3 L0,6 Z" fill="#00e5ff"/>
          </marker>
          <marker id="arrowO" markerWidth="6" markerHeight="6" refX="3" refY="3" orient="auto">
            <path d="M0,0 L6,3 L0,6 Z" fill="#ff6b35"/>
          </marker>
        </defs>

        <!-- Horizontal force -->
        <line x1="415" y1="150" x2="490" y2="150" stroke="#00e5ff" stroke-width="2" marker-end="url(#arrowC)"/>
        <text x="452" y="142" text-anchor="middle" fill="#00e5ff" font-size="10" font-family="JetBrains Mono">Fh (propulsion)</text>

        <!-- Vertical force -->
        <line x1="415" y1="150" x2="415" y2="110" stroke="#ff6b35" stroke-width="2" marker-end="url(#arrowO)"/>
        <text x="470" y="118" fill="#ff6b35" font-size="10" font-family="JetBrains Mono">Fv (normal mod.)</text>

        <!-- Movement arrow -->
        <line x1="60" y1="150" x2="150" y2="150" stroke="#22c55e" stroke-width="2.5" marker-end="url(#arrowC)" opacity="0.8"/>
        <text x="105" y="138" text-anchor="middle" fill="#22c55e" font-size="11" font-family="Space Grotesk" font-weight="600">Net motion</text>

        <!-- Stick/slip labels -->
        <text x="35" y="185" fill="#555570" font-size="9" font-family="JetBrains Mono">STICK</text>
        <text x="100" y="185" fill="#555570" font-size="9" font-family="JetBrains Mono">SLIP</text>
        <text x="165" y="185" fill="#555570" font-size="9" font-family="JetBrains Mono">STICK</text>
      </svg>
    </div>

    <h3>Steering Logic</h3>
    <p>When the ERM spins clockwise, the rotating force vector creates a yaw torque that biases the robot to curve right. Reverse the spin direction — the torque mirrors — robot curves left. Alternate rapidly and the torques cancel: straight line.</p>

    <div class="control-grid">
      <div class="control-card">
        <div class="control-icon">↑</div>
        <div class="control-label">Forward</div>
        <div class="control-cmd">CW ↔ CCW<br>250ms intervals</div>
      </div>
      <div class="control-card">
        <div class="control-icon">↷</div>
        <div class="control-label">Curve Right</div>
        <div class="control-cmd">Sustained CW</div>
      </div>
      <div class="control-card">
        <div class="control-icon">↶</div>
        <div class="control-label">Curve Left</div>
        <div class="control-cmd">Sustained CCW</div>
      </div>
      <div class="control-card">
        <div class="control-icon">⬛</div>
        <div class="control-label">Stop</div>
        <div class="control-cmd">Motor off</div>
      </div>
    </div>
  </div>
</section>

<div class="divider"></div>

<!-- MATH -->
<section id="math">
  <div class="reveal">
    <p class="section-eyebrow">Physics & Mathematics</p>
    <h2>The equations behind it</h2>

    <h3>1. Centrifugal Force</h3>
    <div class="math-section">
      <div class="label">Core force equation</div>
      <p>The eccentric mass spinning at ω rad/s generates centrifugal force proportional to the square of angular velocity.</p>
    </div>
    <div class="eq-box">
      \[ F_c = m_e \cdot \omega^2 \cdot r \qquad \omega = \frac{2\pi \cdot \text{RPM}}{60} \]
    </div>

    <h3>2. Force Decomposition at Tilt Angle θ</h3>
    <div class="eq-box">
      \[ F_{\text{propulsion}} = F_c \sin\theta \qquad F_{\text{normal}} = F_c \cos\theta \]
    </div>
    <p style="color:#555570; font-size:0.85rem;">At θ = 25° (coin ERM optimal tilt): F<sub>prop</sub> = 0.423·F<sub>c</sub>, F<sub>norm</sub> = 0.906·F<sub>c</sub>. Lower than cylinder ERM at 35° but coin motor generates more total F<sub>c</sub> for its size.</p>

    <h3>3. Stick-Slip Condition</h3>
    <p>Robot moves only when propulsion force exceeds static friction:</p>
    <div class="eq-box">
      \[ F_c \sin\theta > \mu_s \left(Mg - F_c \cos\theta\right) \]
    </div>

    <h3>4. Optimal Switching Frequency</h3>
    <div class="eq-box">
      \[ f_{\text{opt}} = \frac{1}{2\tau_m} \approx 2\text{–}4 \text{ Hz} \quad (250\text{–}500\text{ ms per phase}) \]
    </div>
    <p style="color:#555570; font-size:0.85rem;">Confirmed by SimoBot paper: 500ms CW / 500ms CCW = straight-line motion.</p>

    <h3>5. Yaw Torque (Steering)</h3>
    <div class="eq-box">
      \[ \tau_{\text{yaw}} = F_c \cdot d_{\text{offset}} \]
    </div>
    <p style="color:#555570; font-size:0.85rem;">Reversing ω reverses τ<sub>yaw</sub> sign → opposite curve direction. This is the entire steering system.</p>
  </div>
</section>

<div class="divider"></div>

<!-- HARDWARE -->
<section id="hardware">
  <div class="reveal">
    <p class="section-eyebrow">Bill of Materials</p>
    <h2>Hardware</h2>

    <table class="hw-table">
      <thead>
        <tr><th>Component</th><th>Part</th><th>Purpose</th></tr>
      </thead>
      <tbody>
        <tr><td>MCU</td><td><code>Ai-WB2-M1-I</code></td><td>BL602 — BLE 5.0 + WiFi brain</td></tr>
        <tr><td>H-Bridge</td><td><code>CP6208DTR</code></td><td>Motor CW/CCW direction control</td></tr>
        <tr><td>Boost</td><td><code>HT7737C SOT-89</code></td><td>3.7V → 5V (compensates BJT drop)</td></tr>
        <tr><td>Motor</td><td>Coin ERM 10mm, 3V</td><td>Stick-slip locomotion, low noise</td></tr>
        <tr><td>Battery</td><td>80mAh 20-30C LiPo</td><td>Power (~40min runtime)</td></tr>
        <tr><td>Contacts</td><td>5× Pen nib balls</td><td>Pentagon omnidirectional ground contact</td></tr>
        <tr><td>Cap</td><td>100µF</td><td>Motor inrush buffer</td></tr>
        <tr><td>Cap</td><td>0.1µF</td><td>CP6208 VCC bypass</td></tr>
      </tbody>
    </table>

    <h3>Power Chain</h3>
    <pre><code><span class="cm">// Motor power (boosted)</span>
LiPo 3.7V → HT7737C → 5V → CP6208 → Motor (gets ~3.4V)

<span class="cm">// MCU power (direct)</span>
LiPo 3.7V → BL602 VCC (onboard LDO → 3.3V)

<span class="cm">// Motor control pins</span>
BL602 IO3 (Pin19) → CP6208 AIN → CW when HIGH
BL602 IO1 (Pin26) → CP6208 BIN → CCW when HIGH</code></pre>
  </div>
</section>

<div class="divider"></div>

<!-- BUILD STATUS -->
<section id="build">
  <div class="reveal">
    <p class="section-eyebrow">Development Log</p>
    <h2>Build Status</h2>

    <ul class="status-list">
      <li><span class="dot done"></span> BL602 module UART communication confirmed</li>
      <li><span class="dot done"></span> BLE advertising verified (AXK_AT firmware)</li>
      <li><span class="dot done"></span> CP6208 wired and bench tested</li>
      <li><span class="dot done"></span> ERM motor CW/CCW confirmed with manual wire test</li>
      <li><span class="dot active"></span> Custom GPIO firmware flashing in progress</li>
      <li><span class="dot pending"></span> Locomotion direction hypothesis test</li>
      <li><span class="dot pending"></span> Pentagon pen nib chassis assembly</li>
      <li><span class="dot pending"></span> BLE phone control app</li>
      <li><span class="dot pending"></span> Full robot integration test</li>
    </ul>

    <h3>Firmware — Control Logic</h3>
    <pre><code><span class="cm">// BLE command handler — BL602 custom firmware</span>
<span class="kw">void</span> <span class="fn">handle_ble_command</span>(<span class="kw">uint8_t</span> cmd) {
    <span class="kw">switch</span>(cmd) {
        <span class="kw">case</span> <span class="str">'F'</span>: <span class="fn">forward</span>();  <span class="kw">break</span>; <span class="cm">// CW↔CCW 250ms</span>
        <span class="kw">case</span> <span class="str">'R'</span>: <span class="fn">right</span>();    <span class="kw">break</span>; <span class="cm">// Sustained CW</span>
        <span class="kw">case</span> <span class="str">'L'</span>: <span class="fn">left</span>();     <span class="kw">break</span>; <span class="cm">// Sustained CCW</span>
        <span class="kw">case</span> <span class="str">'S'</span>: <span class="fn">stop</span>();     <span class="kw">break</span>; <span class="cm">// Motor off</span>
    }
}

<span class="kw">void</span> <span class="fn">forward</span>() {
    <span class="kw">while</span>(moving) {
        <span class="fn">gpio_write</span>(IO3, HIGH); <span class="fn">gpio_write</span>(IO1, LOW);
        <span class="fn">delay</span>(250);
        <span class="fn">gpio_write</span>(IO3, LOW);  <span class="fn">gpio_write</span>(IO1, HIGH);
        <span class="fn">delay</span>(250);
    }
}</code></pre>
  </div>
</section>

<footer>
  Built by <a href="https://github.com/Ishu1519">Ishant Jaiswal</a> · B.Tech Robotics & Automation ·
  Validated by SimoBot &amp; Pro Know Robot research ·
  <a href="https://github.com/Ishu1519">GitHub →</a>
</footer>

<script>
// ── Oscilloscope animation ──
const canvas = document.getElementById('oscCanvas');
const ctx = canvas.getContext('2d');
let t = 0;

function drawOsc() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  ctx.strokeStyle = '#00e5ff';
  ctx.lineWidth = 2;
  ctx.shadowColor = '#00e5ff';
  ctx.shadowBlur = 8;
  ctx.beginPath();

  for (let x = 0; x < canvas.width; x++) {
    const phase = (x / canvas.width) * Math.PI * 8 + t;
    // Simulate CW/CCW switching — alternating sine with direction flip
    const period = Math.floor((x / canvas.width) * 8);
    const direction = period % 2 === 0 ? 1 : -1;
    const decay = Math.sin(((x / canvas.width) * 8 % 1) * Math.PI);
    const y = canvas.height / 2 + direction * decay * 40 * Math.sin(phase * 3);

    if (x === 0) ctx.moveTo(x, y);
    else ctx.lineTo(x, y);
  }
  ctx.stroke();

  // Phase labels
  ctx.font = '10px JetBrains Mono, monospace';
  ctx.fillStyle = 'rgba(0,229,255,0.5)';
  ctx.shadowBlur = 0;
  for (let i = 0; i < 4; i++) {
    const x = (i / 4) * canvas.width + canvas.width / 8;
    ctx.fillText(i % 2 === 0 ? 'CW' : 'CCW', x, 14);
  }

  t += 0.03;
  requestAnimationFrame(drawOsc);
}
drawOsc();

// ── Scroll reveal ──
const reveals = document.querySelectorAll('.reveal');
const observer = new IntersectionObserver((entries) => {
  entries.forEach(e => {
    if (e.isIntersecting) e.target.classList.add('visible');
  });
}, { threshold: 0.15 });
reveals.forEach(r => observer.observe(r));

// ── Eccentric mass orbit animation ──
const eccMass = document.getElementById('eccMass');
let angle = 0;
function animateMass() {
  if (eccMass) {
    const r = 6;
    const cx = 0, cy = -25;
    const x = cx + r * Math.cos(angle);
    const y = cy + r * Math.sin(angle);
    eccMass.setAttribute('cx', x);
    eccMass.setAttribute('cy', y);
    angle += 0.08;
  }
  requestAnimationFrame(animateMass);
}
animateMass();
</script>

</body>
</html>
