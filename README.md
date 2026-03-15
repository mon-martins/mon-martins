
<style>
  @import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;700&display=swap');
  * { box-sizing: border-box; margin: 0; padding: 0; }

  .banner {
    width: 100%;
    background: #0d1117;
    border-radius: 12px;
    overflow: hidden;
    font-family: 'JetBrains Mono', monospace;
    position: relative;
    border: 1px solid #21262d;
  }

  .terminal {
    position: relative; z-index: 2;
    margin: 24px 24px 24px;
    background: #010409;
    border: 1px solid #30363d;
    border-radius: 8px;
    overflow: hidden;
  }

  .term-header {
    background: #161b22;
    padding: 9px 14px;
    display: flex;
    align-items: center;
    gap: 7px;
    border-bottom: 1px solid #21262d;
  }
  .dot { width: 11px; height: 11px; border-radius: 50%; }
  .d-red    { background: #ff5f57; }
  .d-yellow { background: #febc2e; }
  .d-green  { background: #28c840; }
  .term-title { color: #6e7681; font-size: 11px; margin-left: 6px; letter-spacing: 0.5px; }

  .term-body {
    padding: 14px 18px;
    height: 450px;
    overflow: hidden;
    font-size: 12px;
    line-height: 1.65;
    color: #e6edf3;
  }

  .line { display: flex; align-items: flex-start; flex-wrap: wrap; margin: 1px 0; }
  .out      { color: #8b949e; display: block; margin: 1px 0; }
  .out-ok   { color: #3fb950; display: block; margin: 1px 0; }
  .out-warn { color: #d29922; display: block; margin: 1px 0; }
  .out-run  { color: #58a6ff; display: block; margin: 1px 0; }
  .out-err  { color: #f85149; display: block; margin: 1px 0; }
  .out-test { display: block; margin: 1px 0; }
  .test-pass { color: #3fb950; }
  .test-name { color: #e6edf3; }

  .cursor {
    display: inline-block; width: 7px; height: 13px;
    background: #58a6ff;
    animation: blink 1s step-end infinite;
    vertical-align: bottom; margin-left: 1px;
  }
  @keyframes blink { 0%,100%{opacity:1} 50%{opacity:0} }

  .fade-in { animation: fadeIn 0.12s ease-in; }
  @keyframes fadeIn { from{opacity:0} to{opacity:1} }
</style>

<div class="banner" id="banner">
  <div class="terminal">
    <div class="term-header">
      <div class="dot d-red"></div>
      <div class="dot d-yellow"></div>
      <div class="dot d-green"></div>
      <span class="term-title">bash — ~/projects/ControlECU</span>
    </div>
    <div class="term-body" id="term-body"></div>
  </div>
</div>

<script>
  const tb = document.getElementById('term-body');
  const BRANCH = 'feat-flc_control';

  const script = [
    { type: 'cmd', prompt: true,  text: 'docker run -it --rm -v $(pwd):/workspace unit-test-env bash' },
    { type: 'out', text: 'latest: Pulling from unit-test-env ... Done', cls: 'out-warn' },
    { type: 'out', text: '✔ Container started · /workspace mounted',    cls: 'out-ok'  },
    { type: 'cmd', prompt: false, dPrompt: 'root@container:/workspace#', text: './build.sh' },
    { type: 'out', text: '[1/5] Generating state-machine code',                       cls: 'out-run' },
    { type: 'out', text: '[2/5] Generating C code for ecucontrol.dbc',                cls: 'out-run' },
    { type: 'out', text: '[3/5] Generating peripheral configuration from ecu.syscfg', cls: 'out-run' },
    { type: 'out', text: '[4/5] Compiling   src/ ...',                                cls: 'out-run' },
    { type: 'out', text: '[5/5] Linking     ControlECU.elf  (112.6 KB)',              cls: 'out-run' },
    { type: 'out', text: '✔ BUILD SUCCESSFUL — 0 errors, 0 warnings',                cls: 'out-ok'  },
    { type: 'cmd', prompt: false, dPrompt: 'root@container:/workspace#', text: './unit_test.sh' },
    { type: 'out', text: '──────────────────────────────────────────────────', cls: 'out' },
    { type: 'test', name: 'blinky_turn_off_led' },
    { type: 'test', name: 'blinky_toggle_led_on_event' },
    { type: 'test', name: 'blinky_init_state_is_idle' },
    { type: 'test', name: 'communication_send_can_message' },
    { type: 'test', name: 'communication_receive_can_frame' },
    { type: 'test', name: 'communication_handle_bus_off_error' },
    { type: 'test', name: 'control_change_duty_cycle' },
    { type: 'test', name: 'control_nonlinear_output_within_bounds' },
    { type: 'test', name: 'control_ramp_up_follows_reference' },
    { type: 'test', name: 'control_saturation_clamps_output' },
    { type: 'out', text: '──────────────────────────────────────────────────', cls: 'out' },
    { type: 'out', text: '✔ Tests: 10 passed · 0 failed · 0 skipped', cls: 'out-ok' },
    { type: 'cmd', prompt: false, dPrompt: 'root@container:/workspace#', text: 'exit' },
    { type: 'out', text: '✔ Exited container', cls: 'out-ok' },
    { type: 'cmd', prompt: true, text: 'git add .' },
    { type: 'cmd', prompt: true, text: 'git commit -m "add: new non-linear control ready for tests on HIL"' },
    { type: 'out', text: '[feat-flc_control 7c2d1a3] add: new non-linear control ready for tests on HIL', cls: 'out' },
    { type: 'out', text: ' 5 files changed, 218 insertions(+), 11 deletions(-)', cls: 'out' },
    { type: 'cmd', prompt: true, text: 'git push' },
    { type: 'out', text: 'fatal: The current branch feat-flc_control has no upstream branch.', cls: 'out-err' },
    { type: 'out', text: '       To push the current branch and set the remote as upstream, use:', cls: 'out-err' },
    { type: 'out', text: '       git push --set-upstream origin feat-flc_control', cls: 'out-err' },
    { type: 'cmd', prompt: true, text: 'git push --set-upstream origin feat-flc_control' },
    { type: 'out', text: 'Enumerating objects: 8, done.',                        cls: 'out'    },
    { type: 'out', text: 'Counting objects: 100% (8/8), done.',                  cls: 'out'    },
    { type: 'out', text: 'To github.com:ramon/ControlECU.git',                   cls: 'out'    },
    { type: 'out', text: ' * [new branch]  feat-flc_control → feat-flc_control', cls: 'out-ok' },
    { type: 'out', text: "Branch 'feat-flc_control' set up to track remote branch.", cls: 'out-ok' },
    { type: 'out', text: '✔ Push successful',                                    cls: 'out-ok' },
  ];

  const CHAR_DELAY = 34;
  const LOOP_PAUSE = 3200;

  function makePromptHTML(dPrompt) {
    if (dPrompt) return `<span style="color:#f0883e;">${dPrompt}</span> `;
    return `<span style="color:#3fb950;">ramon</span><span style="color:#8b949e;">@</span><span style="color:#58a6ff;">dev</span><span style="color:#8b949e;">:</span><span style="color:#f0883e;">~/projects/ControlECU</span><span style="color:#8b949e;"> (</span><span style="color:#3fb950;">${BRANCH}</span><span style="color:#8b949e;">)</span><span style="color:#e6edf3;">$</span> `;
  }

  function sleep(ms) { return new Promise(r => setTimeout(r, ms)); }
  function scrollToBottom() { tb.scrollTop = tb.scrollHeight; }

  async function typeText(cmdEl, text, delay) {
    for (const ch of text) {
      cmdEl.textContent += ch;
      scrollToBottom();
      await sleep(delay);
    }
  }

  async function runScript() {
    tb.innerHTML = '';
    const cursor = document.createElement('span');
    cursor.className = 'cursor';
    tb.appendChild(cursor);
    await sleep(500);

    for (const step of script) {
      if (step.type === 'cmd') {
        cursor.remove();
        const line = document.createElement('div');
        line.className = 'line fade-in';
        line.innerHTML = makePromptHTML(step.dPrompt);
        const cmdEl = document.createElement('span');
        cmdEl.style.color = '#e6edf3';
        line.appendChild(cmdEl);
        line.appendChild(cursor);
        tb.appendChild(line);
        scrollToBottom();
        await sleep(280);
        await typeText(cmdEl, step.text, CHAR_DELAY);
        cursor.remove();
        await sleep(700);

      } else if (step.type === 'out') {
        const span = document.createElement('span');
        span.className = (step.cls || 'out') + ' fade-in';
        span.textContent = step.text;
        tb.appendChild(span);
        scrollToBottom();
        await sleep(85);

      } else if (step.type === 'test') {
        const span = document.createElement('span');
        span.className = 'out-test fade-in';
        span.innerHTML = `<span class="test-pass">  ✔ PASS </span><span class="test-name">${step.name}</span>`;
        tb.appendChild(span);
        scrollToBottom();
        await sleep(70);
      }
    }

    const finalLine = document.createElement('div');
    finalLine.className = 'line';
    finalLine.innerHTML = makePromptHTML();
    finalLine.appendChild(cursor);
    tb.appendChild(finalLine);
    scrollToBottom();

    await sleep(LOOP_PAUSE);
    runScript();
  }

  runScript();
</script>
