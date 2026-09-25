(function() {
  'use strict';
  
  // Prevent duplicate injections
  if (window.__autoClickerActive) {
    console.log('[Auto Clicker] Already active');
    return;
  }
  window.__autoClickerActive = true;
  
  // State management
  let clickInterval = null;
  let targetX = null;
  let targetY = null;
  let isRunning = false;
  let isPickingPosition = false;
  let clickCount = 0;
  let sessionStart = null;
  
  // Create main container
  const container = document.createElement('div');
  container.id = 'auto-clicker-ui';
  container.style.cssText = `
    position: fixed;
    top: 50px;
    right: 50px;
    width: 320px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    border-radius: 12px;
    box-shadow: 0 8px 32px rgba(0,0,0,0.4);
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    z-index: 2147483647;
    user-select: none;
    overflow: hidden;
    animation: slideIn 0.3s ease-out;
  `;
  
  // Add keyframe animation
  const style = document.createElement('style');
  style.textContent = `
    @keyframes slideIn {
      from {
        transform: translateX(400px);
        opacity: 0;
      }
      to {
        transform: translateX(0);
        opacity: 1;
      }
    }
    @keyframes pulse {
      0%, 100% { transform: scale(1); }
      50% { transform: scale(1.05); }
    }
    #auto-clicker-ui input:focus {
      outline: 2px solid #fff;
      outline-offset: 2px;
    }
  `;
  document.head.appendChild(style);
  
  // Header (draggable)
  const header = document.createElement('div');
  header.style.cssText = `
    background: rgba(0,0,0,0.2);
    padding: 14px 16px;
    cursor: move;
    display: flex;
    justify-content: space-between;
    align-items: center;
    color: white;
    font-weight: 600;
    backdrop-filter: blur(10px);
  `;
  header.innerHTML = `
    <div style="display: flex; align-items: center; gap: 8px;">
      <span style="font-size: 18px;">⚡</span>
      <span style="font-size: 14px;">Auto Clicker Pro</span>
    </div>
    <div style="display: flex; gap: 6px;">
      <button id="ac-minimize" style="
        background: rgba(255,255,255,0.2);
        border: none;
        color: white;
        width: 26px;
        height: 26px;
        border-radius: 6px;
        cursor: pointer;
        font-weight: bold;
        font-size: 16px;
        transition: all 0.2s;
      ">−</button>
      <button id="ac-close" style="
        background: rgba(231,76,60,0.8);
        border: none;
        color: white;
        width: 26px;
        height: 26px;
        border-radius: 6px;
        cursor: pointer;
        font-weight: bold;
        font-size: 16px;
        transition: all 0.2s;
      ">×</button>
    </div>
  `;
  
  // Content area
  const content = document.createElement('div');
  content.id = 'ac-content';
  content.style.cssText = `
    padding: 18px;
    color: white;
    background: rgba(255,255,255,0.1);
    backdrop-filter: blur(20px);
  `;
  
  content.innerHTML = `
    <!-- Statistics -->
    <div style="
      background: rgba(0,0,0,0.2);
      padding: 12px;
      border-radius: 8px;
      margin-bottom: 16px;
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      font-size: 12px;
    ">
      <div>
        <div style="opacity: 0.8;">Total Clicks</div>
        <div id="ac-click-count" style="font-size: 20px; font-weight: bold;">0</div>
      </div>
      <div>
        <div style="opacity: 0.8;">Session Time</div>
        <div id="ac-session-time" style="font-size: 20px; font-weight: bold;">00:00</div>
      </div>
    </div>
    
    <!-- CPS Control -->
    <div style="margin-bottom: 16px;">
      <label style="display: flex; justify-content: space-between; margin-bottom: 8px; font-size: 13px; font-weight: 500;">
        <span>Clicks Per Second</span>
        <span id="ac-cps-display" style="background: rgba(0,0,0,0.3); padding: 2px 8px; border-radius: 4px;">10</span>
      </label>
      <input 
        id="ac-cps" 
        type="range" 
        min="1" 
        max="100" 
        value="10"
        style="
          width: 100%;
          height: 6px;
          border-radius: 3px;
          background: rgba(255,255,255,0.3);
          outline: none;
          -webkit-appearance: none;
        "
      />
      <style>
        #ac-cps::-webkit-slider-thumb {
          -webkit-appearance: none;
          width: 18px;
          height: 18px;
          border-radius: 50%;
          background: white;
          cursor: pointer;
          box-shadow: 0 2px 6px rgba(0,0,0,0.3);
        }
        #ac-cps::-moz-range-thumb {
          width: 18px;
          height: 18px;
          border-radius: 50%;
          background: white;
          cursor: pointer;
          border: none;
          box-shadow: 0 2px 6px rgba(0,0,0,0.3);
        }
      </style>
    </div>
    
    <!-- Click Mode -->
    <div style="margin-bottom: 16px;">
      <label style="display: block; margin-bottom: 8px; font-size: 13px; font-weight: 500;">
        Click Mode
      </label>
      <div style="display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 6px;">
        <button class="ac-mode-btn" data-mode="left" style="
          padding: 8px;
          background: rgba(255,255,255,0.9);
          border: 2px solid transparent;
          color: #667eea;
          border-radius: 6px;
          cursor: pointer;
          font-weight: 600;
          font-size: 12px;
          transition: all 0.2s;
        ">Left</button>
        <button class="ac-mode-btn" data-mode="middle" style="
          padding: 8px;
          background: rgba(255,255,255,0.2);
          border: 2px solid transparent;
          color: white;
          border-radius: 6px;
          cursor: pointer;
          font-weight: 600;
          font-size: 12px;
          transition: all 0.2s;
        ">Middle</button>
        <button class="ac-mode-btn" data-mode="right" style="
          padding: 8px;
          background: rgba(255,255,255,0.2);
          border: 2px solid transparent;
          color: white;
          border-radius: 6px;
          cursor: pointer;
          font-weight: 600;
          font-size: 12px;
          transition: all 0.2s;
        ">Right</button>
      </div>
    </div>
    
    <!-- Position Picker -->
    <div style="margin-bottom: 16px;">
      <button id="ac-pick" style="
        width: 100%;
        padding: 12px;
        background: rgba(255,255,255,0.2);
        border: 2px dashed rgba(255,255,255,0.5);
        color: white;
        border-radius: 8px;
        cursor: pointer;
        font-weight: 600;
        font-size: 13px;
        transition: all 0.2s;
      ">📍 Pick Target Position</button>
      <div id="ac-coords" style="
        margin-top: 8px;
        font-size: 11px;
        color: rgba(255,255,255,0.7);
        text-align: center;
        font-family: monospace;
      ">No position selected</div>
    </div>
    
    <!-- Repeat Options -->
    <div style="margin-bottom: 16px;">
      <label style="display: flex; align-items: center; gap: 8px; font-size: 13px; margin-bottom: 8px;">
        <input type="checkbox" id="ac-repeat-limit" style="width: 16px; height: 16px; cursor: pointer;">
        <span>Stop after</span>
        <input 
          id="ac-repeat-count" 
          type="number" 
          min="1" 
          value="100" 
          disabled
          style="
            width: 70px;
            padding: 4px 8px;
            background: rgba(0,0,0,0.2);
            border: 1px solid rgba(255,255,255,0.3);
            color: white;
            border-radius: 4px;
            font-size: 12px;
          "
        />
        <span>clicks</span>
      </label>
    </div>
    
    <!-- Control Buttons -->
    <button id="ac-toggle" style="
      width: 100%;
      padding: 14px;
      background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
      border: none;
      color: white;
      border-radius: 8px;
      cursor: pointer;
      font-weight: 700;
      font-size: 15px;
      transition: all 0.2s;
      box-shadow: 0 4px 15px rgba(56,239,125,0.3);
      text-transform: uppercase;
      letter-spacing: 1px;
    ">▶ Start Clicking</button>
    
    <!-- Hotkey Info -->
    <div style="
      margin-top: 12px;
      padding: 10px;
      background: rgba(0,0,0,0.2);
      border-radius: 6px;
      font-size: 11px;
      text-align: center;
      opacity: 0.8;
    ">
      💡 Press <kbd style="background: rgba(255,255,255,0.2); padding: 2px 6px; border-radius: 3px;">ESC</kbd> to stop clicking
    </div>
  `;
  
  container.appendChild(header);
  container.appendChild(content);
  document.body.appendChild(container);
  
  // Get elements
  const closeBtn = document.getElementById('ac-close');
  const minimizeBtn = document.getElementById('ac-minimize');
  const cpsInput = document.getElementById('ac-cps');
  const cpsDisplay = document.getElementById('ac-cps-display');
  const pickBtn = document.getElementById('ac-pick');
  const coordsDisplay = document.getElementById('ac-coords');
  const toggleBtn = document.getElementById('ac-toggle');
  const clickCountDisplay = document.getElementById('ac-click-count');
  const sessionTimeDisplay = document.getElementById('ac-session-time');
  const repeatLimitCheckbox = document.getElementById('ac-repeat-limit');
  const repeatCountInput = document.getElementById('ac-repeat-count');
  const modeButtons = document.querySelectorAll('.ac-mode-btn');
  
  let clickMode = 'left';
  let sessionTimer = null;
  
  // Mode selection
  modeButtons.forEach(btn => {
    btn.addEventListener('click', function() {
      modeButtons.forEach(b => {
        b.style.background = 'rgba(255,255,255,0.2)';
        b.style.color = 'white';
      });
      this.style.background = 'rgba(255,255,255,0.9)';
      this.style.color = '#667eea';
      clickMode = this.dataset.mode;
    });
  });
  
  // CPS slider
  cpsInput.addEventListener('input', (e) => {
    cpsDisplay.textContent = e.target.value;
    if (isRunning) {
      stopClicking();
      startClicking();
    }
  });
  
  // Repeat limit checkbox
  repeatLimitCheckbox.addEventListener('change', (e) => {
    repeatCountInput.disabled = !e.target.checked;
  });
  
  // Dragging functionality
  let isDragging = false;
  let offsetX = 0;
  let offsetY = 0;
  
  header.addEventListener('mousedown', (e) => {
    if (e.target.id === 'ac-close' || e.target.id === 'ac-minimize') return;
    isDragging = true;
    offsetX = e.clientX - container.offsetLeft;
    offsetY = e.clientY - container.offsetTop;
    header.style.cursor = 'grabbing';
  });
  
  document.addEventListener('mousemove', (e) => {
    if (!isDragging) return;
    container.style.left = (e.clientX - offsetX) + 'px';
    container.style.top = (e.clientY - offsetY) + 'px';
    container.style.right = 'auto';
  });
  
  document.addEventListener('mouseup', () => {
    isDragging = false;
    header.style.cursor = 'move';
  });
  
  // Minimize functionality
  let isMinimized = false;
  minimizeBtn.addEventListener('click', () => {
    isMinimized = !isMinimized;
    if (isMinimized) {
      content.style.display = 'none';
      minimizeBtn.textContent = '+';
      container.style.width = '200px';
    } else {
      content.style.display = 'block';
      minimizeBtn.textContent = '−';
      container.style.width = '320px';
    }
  });
  
  // Close button
  closeBtn.addEventListener('click', () => {
    stopClicking();
    container.style.animation = 'slideIn 0.3s ease-out reverse';
    setTimeout(() => {
      container.remove();
      style.remove();
      window.__autoClickerActive = false;
    }, 300);
  });
  
  // Pick position functionality
  pickBtn.addEventListener('click', () => {
    if (isPickingPosition) return;
    
    isPickingPosition = true;
    pickBtn.textContent = '🎯 Click anywhere on the page...';
    pickBtn.style.background = 'rgba(230,126,34,0.8)';
    pickBtn.style.animation = 'pulse 1s infinite';
    document.body.style.cursor = 'crosshair';
    
    const overlay = document.createElement('div');
    overlay.style.cssText = `
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0,0,0,0.1);
      z-index: 2147483646;
      cursor: crosshair;
    `;
    document.body.appendChild(overlay);
    
    const pickHandler = (e) => {
      e.preventDefault();
      e.stopPropagation();
      
      targetX = e.clientX;
      targetY = e.clientY;
      
      coordsDisplay.textContent = `✓ Target locked: X=${targetX}, Y=${targetY}`;
      coordsDisplay.style.color = '#38ef7d';
      
      pickBtn.textContent = '📍 Pick Target Position';
      pickBtn.style.background = 'rgba(255,255,255,0.2)';
      pickBtn.style.animation = 'none';
      document.body.style.cursor = 'default';
      isPickingPosition = false;
      
      overlay.remove();
      document.removeEventListener('click', pickHandler, true);
    };
    
    document.addEventListener('click', pickHandler, true);
  });
  
  // Session timer
  function updateSessionTime() {
    if (!sessionStart) return;
    const elapsed = Math.floor((Date.now() - sessionStart) / 1000);
    const minutes = Math.floor(elapsed / 60).toString().padStart(2, '0');
    const seconds = (elapsed % 60).toString().padStart(2, '0');
    sessionTimeDisplay.textContent = `${minutes}:${seconds}`;
  }
  
  // Auto-click functionality
  function performClick() {
    if (targetX === null || targetY === null) return;
    
    const element = document.elementFromPoint(targetX, targetY);
    if (!element) return;
    
    let button = 0;
    if (clickMode === 'middle') button = 1;
    if (clickMode === 'right') button = 2;
    
    const eventOptions = {
      bubbles: true,
      cancelable: true,
      view: window,
      clientX: targetX,
      clientY: targetY,
      button: button
    };
    
    element.dispatchEvent(new MouseEvent('mousedown', eventOptions));
    element.dispatchEvent(new MouseEvent('mouseup', eventOptions));
    element.dispatchEvent(new MouseEvent('click', eventOptions));
    
    clickCount++;
    clickCountDisplay.textContent = clickCount.toLocaleString();
    
    // Check repeat limit
    if (repeatLimitCheckbox.checked) {
      const limit = parseInt(repeatCountInput.value);
      if (clickCount >= limit) {
        stopClicking();
        alert(`Auto Clicker stopped: Reached ${limit} clicks`);
      }
    }
  }
  
  function startClicking() {
    if (targetX === null || targetY === null) {
      alert('⚠️ Please pick a position first!');
      return;
    }
    
    const cps = parseInt(cpsInput.value) || 10;
    const interval = 1000 / cps;
    
    isRunning = true;
    sessionStart = Date.now();
    clickCount = 0;
    
    toggleBtn.textContent = '⏸ Stop Clicking';
    toggleBtn.style.background = 'linear-gradient(135deg, #eb3349 0%, #f45c43 100%)';
    toggleBtn.style.boxShadow = '0 4px 15px rgba(235,51,73,0.4)';
    
    clickInterval = setInterval(performClick, interval);
    sessionTimer = setInterval(updateSessionTime, 1000);
  }
  
  function stopClicking() {
    isRunning = false;
    
    toggleBtn.textContent = '▶ Start Clicking';
    toggleBtn.style.background = 'linear-gradient(135deg, #11998e 0%, #38ef7d 100%)';
    toggleBtn.style.boxShadow = '0 4px 15px rgba(56,239,125,0.3)';
    
    if (clickInterval) {
      clearInterval(clickInterval);
      clickInterval = null;
    }
    
    if (sessionTimer) {
      clearInterval(sessionTimer);
      sessionTimer = null;
    }
  }
  
  // Toggle button
  toggleBtn.addEventListener('click', () => {
    if (isRunning) {
      stopClicking();
    } else {
      startClicking();
    }
  });
  
  // ESC key to stop
  document.addEventListener('keydown', (e) => {
    if (e.key === 'Escape' && isRunning) {
      stopClicking();
    }
  });
  
  // Hover effects
  pickBtn.addEventListener('mouseenter', function() {
    if (!isPickingPosition) {
      this.style.background = 'rgba(255,255,255,0.3)';
    }
  });
  
  pickBtn.addEventListener('mouseleave', function() {
    if (!isPickingPosition) {
      this.style.background = 'rgba(255,255,255,0.2)';
    }
  });
  
  toggleBtn.addEventListener('mouseenter', function() {
    this.style.transform = 'translateY(-2px)';
    this.style.boxShadow = isRunning 
      ? '0 6px 20px rgba(235,51,73,0.5)' 
      : '0 6px 20px rgba(56,239,125,0.4)';
  });
  
  toggleBtn.addEventListener('mouseleave', function() {
    this.style.transform = 'translateY(0)';
    this.style.boxShadow = isRunning 
      ? '0 4px 15px rgba(235,51,73,0.4)' 
      : '0 4px 15px rgba(56,239,125,0.3)';
  });
  
  closeBtn.addEventListener('mouseenter', function() {
    this.style.background = 'rgba(192,57,43,1)';
    this.style.transform = 'scale(1.1)';
  });
  
  closeBtn.addEventListener('mouseleave', function() {
    this.style.background = 'rgba(231,76,60,0.8)';
    this.style.transform = 'scale(1)';
  });
  
  minimizeBtn.addEventListener('mouseenter', function() {
    this.style.background = 'rgba(255,255,255,0.3)';
    this.style.transform = 'scale(1.1)';
  });
  
  minimizeBtn.addEventListener('mouseleave', function() {
    this.style.background = 'rgba(255,255,255,0.2)';
    this.style.transform = 'scale(1)';
  });
  
  console.log('[Auto Clicker Pro] Loaded successfully! 🚀');
})();
