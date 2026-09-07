<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>DÙNG ĐƯỢC? — USABLE ARTWORK</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; }
    body {
      background-color: #0a0a0a;
      color: #fff;
      font-family: monospace;
      height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: space-between;
      padding: 15px;
      overflow: hidden;
    }

    /* Thanh chọn vật dụng */
    #nav-bar {
      display: flex;
      gap: 10px;
      z-index: 10;
      background: rgba(20, 20, 20, 0.8);
      padding: 8px 12px;
      border-radius: 20px;
      border: 1px solid #333;
    }

    .nav-btn {
      background: transparent;
      border: none;
      color: #888;
      font-family: monospace;
      font-size: 0.8rem;
      cursor: pointer;
      padding: 4px 8px;
      border-radius: 12px;
      transition: all 0.3s;
    }

    .nav-btn.active {
      color: #00ff66;
      background: #222;
      font-weight: bold;
    }

    /* Sân khấu mô phỏng 3D CSS */
    #viewport {
      perspective: 1000px;
      width: 260px;
      height: 320px;
      position: relative;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .object-container {
      width: 100%;
      height: 100%;
      position: absolute;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: opacity 0.5s ease;
    }

    /* 01. CỬA */
    #door-frame { width: 180px; height: 280px; border: 4px solid #333; background: #111; position: relative; }
    #door-leaf {
      width: 100%; height: 100%; background: #222; border: 1px solid #444;
      transform-origin: left; transition: transform 1.2s cubic-bezier(0.4, 0, 0.2, 1);
      display: flex; align-items: center; justify-content: flex-end; padding-right: 15px;
    }

    /* 02. VÒI NƯỚC */
    #faucet-body { width: 40px; height: 140px; background: #333; border-radius: 20px 20px 0 0; position: relative; display: flex; flex-direction: column; align-items: center; }
    #faucet-spout { width: 80px; height: 20px; background: #333; position: absolute; top: 20px; right: -60px; border-radius: 0 10px 10px 0; }
    #water-stream { width: 12px; height: 0px; background: #00ccff; position: absolute; top: 40px; right: -56px; transition: height 0.5s; opacity: 0.8; }

    /* 03. CÔNG TẮC ĐÈN */
    #switch-plate { width: 160px; height: 160px; background: #222; border: 2px solid #444; border-radius: 12px; display: flex; align-items: center; justify-content: center; position: relative; }
    #light-bulb { width: 30px; height: 30px; border-radius: 50%; background: #333; position: absolute; top: -50px; transition: background 0.3s; }

    /* MÔ-ĐUN TAY CẦM / ĐIỀU KHIỂN BIẾN HÌNH */
    .module-mesh {
      background: #00ccff;
      border-radius: 4px;
      transition: all 0.6s cubic-bezier(0.68, -0.55, 0.265, 1.55);
      box-shadow: 0 0 12px rgba(0, 204, 255, 0.6);
    }

    /* Các dạng mô-đun biến hình */
    .mod-knob { width: 22px; height: 22px; border-radius: 50% !important; transform: translate(0, 0); }
    .mod-bar { width: 10px; height: 160px; border-radius: 5px !important; transform: translate(0, 0); }
    .mod-button { width: 50px; height: 50px; border-radius: 10px !important; transform: translate(0, 0); }
    .mod-sensor { width: 120px; height: 120px; border-radius: 50% !important; background: rgba(0, 255, 102, 0.3); border: 2px dashed #00ff66; box-shadow: none; }
    .mod-detached { transform: translate(-40px, -30px) scale(0.6) rotate(45deg); opacity: 0.7; }

    /* Bảng thông báo & Bắt cử chỉ */
    #ui-card {
      background: rgba(20, 20, 20, 0.95);
      border: 1px solid #333;
      padding: 18px;
      border-radius: 10px;
      text-align: center;
      max-width: 420px;
      width: 100%;
    }

    .title { font-size: 1rem; font-weight: bold; color: #00ccff; margin-bottom: 6px; }
    .desc { font-size: 0.8rem; color: #aaa; margin-bottom: 12px; min-height: 36px; }

    .btn-group { display: flex; gap: 6px; justify-content: center; flex-wrap: wrap; }
    button.action-btn {
      background: #222; color: #fff; border: 1px solid #555;
      padding: 6px 10px; font-family: monospace; font-size: 0.75rem;
      border-radius: 4px; cursor: pointer;
    }
    button.action-btn:active { background: #00ff66; color: #000; }

    #wall-of-ways { display: none; text-align: left; }
    .stat-val { font-size: 1.6rem; font-weight: bold; color: #ff3366; }
  </style>
</head>
<body>

  <!-- Thanh chuyển Vật dụng -->
  <div id="nav-bar">
    <button class="nav-btn active" onclick="switchObject('DOOR')">01. CỬA</button>
    <button class="nav-btn" onclick="switchObject('FAUCET')">02. VÒI NƯỚC</button>
    <button class="nav-btn" onclick="switchObject('SWITCH')">03. CÔNG TẮC</button>
  </div>

  <!-- Viewport Đồ họa 3D -->
  <div id="viewport">
    
    <!-- 01. CÁNH CỬA -->
    <div id="obj-door" class="object-container">
      <div id="door-frame">
        <div id="door-leaf">
          <div id="door-handle" class="module-mesh mod-knob"></div>
        </div>
      </div>
    </div>

    <!-- 02. VÒI NƯỚC -->
    <div id="obj-faucet" class="object-container" style="opacity: 0; pointer-events: none;">
      <div id="faucet-body">
        <div id="faucet-spout">
          <div id="water-stream"></div>
        </div>
        <div id="faucet-handle" class="module-mesh mod-knob" style="margin-top: -15px;"></div>
      </div>
    </div>

    <!-- 03. CÔNG TẮC ĐÈN -->
    <div id="obj-switch" class="object-container" style="opacity: 0; pointer-events: none;">
      <div id="light-bulb"></div>
      <div id="switch-plate">
        <div id="switch-handle" class="module-mesh mod-knob"></div>
      </div>
    </div>

  </div>

  <!-- Khung điều khiển & Kịch bản -->
  <div id="ui-card">
    <div id="interactive-panel">
      <div class="title" id="p-title">01 — CÁCH DÙNG QUEN THUỘC</div>
      <div class="desc" id="p-desc">Thao tác vặn/xoay truyền thống. Hãy thử kích hoạt vật dụng.</div>

      <div class="btn-group">
        <button class="action-btn" onclick="gestureInput('ROTATE')">1. Xoay cổ tay</button>
        <button class="action-btn" onclick="gestureInput('PALM')">2. Xòe lòng bàn tay</button>
        <button class="action-btn" onclick="gestureInput('POINT')">3. Chỉ 1 ngón tay</button>
      </div>
    </div>

    <!-- Bức tường DÙNG ĐƯỢC -->
    <div id="wall-of-ways">
      <div style="color:#fff; font-weight:bold; margin-bottom:8px;">BỨC TƯỜNG DÙNG ĐƯỢC (WALL OF WAYS)</div>
      <p style="font-size:0.75rem; color:#888;">HÔM NAY TẠI TRIỂN LÃM:</p>
      <div class="stat-val">183 NGƯỜI THAM GIA</div>
      <div class="stat-val" style="color:#00ff66;">183 CỬ CHỈ MỚI</div>
      <div class="stat-val" style="color:#00ccff;">42 THIẾT KẾ ĐƯỢC TẠO</div>
      <p style="margin-top:8px; font-size:0.7rem; color:#aaa;">"THAY ĐỔI CÁCH BẠN DÙNG. THAY ĐỔI THIẾT KẾ."</p>
    </div>
  </div>

  <script>
    let currentObj = 'DOOR';
    let step = 1; // 1: Quen thuộc, 2: Tách mô-đun, 3: Biến hình theo cử chỉ, 4: Wall of ways
    let isAnimating = false;

    // DOM Elements
    const pTitle = document.getElementById('p-title');
    const pDesc = document.getElementById('p-desc');
    const doorLeaf = document.getElementById('door-leaf');
    const waterStream = document.getElementById('water-stream');
    const lightBulb = document.getElementById('light-bulb');

    const handles = {
      DOOR: document.getElementById('door-handle'),
      FAUCET: document.getElementById('faucet-handle'),
      SWITCH: document.getElementById('switch-handle')
    };

    function switchObject(objType) {
      if (isAnimating) return;
      currentObj = objType;
      step = 1;
      
      // Update Tab Navigation
      document.querySelectorAll('.nav-btn').forEach((btn, idx) => {
        btn.classList.toggle('active', (idx === 0 && objType==='DOOR') || (idx === 1 && objType==='FAUCET') || (idx === 2 && objType==='SWITCH'));
      });

      // Reset Viewport Display
      document.getElementById('obj-door').style.opacity = objType === 'DOOR' ? '1' : '0';
      document.getElementById('obj-faucet').style.opacity = objType === 'FAUCET' ? '1' : '0';
      document.getElementById('obj-switch').style.opacity = objType === 'SWITCH' ? '1' : '0';

      // Reset Object States
      doorLeaf.style.transform = 'rotateY(0deg)';
      waterStream.style.height = '0px';
      lightBulb.style.background = '#333';
      
      Object.values(handles).forEach(h => h.className = "module-mesh mod-knob");

      document.getElementById('interactive-panel').style.display = 'block';
      document.getElementById('wall-of-ways').style.display = 'none';

      updateText();
    }

    function updateText() {
      if (step === 1) {
        pTitle.innerText = `01 — CÁCH DÙNG QUEN THUỘC (${currentObj})`;
        pDesc.innerText = "Sử dụng thao tác xoay cổ tay chuẩn để vận hành vật dụng.";
      } else if (step === 2) {
        pTitle.innerText = "02 — NHƯNG NẾU BẠN KHÔNG THỂ XOAY CỔ TAY?";
        pDesc.innerText = "Cơ cấu đang tự động rã ra thành các mô-đun biến hình...";
      } else if (step === 3) {
        pTitle.innerText = "03 — TỰ TẠO THIẾT KẾ BẰNG CỬ CHỈ CỦA BẠN";
        pDesc.innerText = "Dùng Xòe tay (Tạo thanh trượt lớn) hoặc Chỉ ngón (Tạo nút cảm ứng).";
      } else if (step === 4) {
        document.getElementById('interactive-panel').style.display = 'none';
        document.getElementById('wall-of-ways').style.display = 'block';
      }
    }

    function executeAction() {
      if (currentObj === 'DOOR') {
        doorLeaf.style.transform = 'rotateY(-105deg)';
        setTimeout(() => doorLeaf.style.transform = 'rotateY(0deg)', 1600);
      } else if (currentObj === 'FAUCET') {
        waterStream.style.height = '120px';
        setTimeout(() => waterStream.style.height = '0px', 1600);
      } else if (currentObj === 'SWITCH') {
        lightBulb.style.background = '#ffff00';
        lightBulb.style.boxShadow = '0 0 20px #ffff00';
        setTimeout(() => {
          lightBulb.style.background = '#333';
          lightBulb.style.boxShadow = 'none';
        }, 1600);
      }
    }

    function gestureInput(gesture) {
      if (isAnimating) return;

      if (step === 1 && gesture === 'ROTATE') {
        isAnimating = true;
        executeAction();

        setTimeout(() => {
          step = 2;
          updateText();
          handles[currentObj].className = "module-mesh mod-detached";

          setTimeout(() => {
            step = 3;
            updateText();
            isAnimating = false;
          }, 2000);
        }, 1800);

      } else if (step === 3) {
        isAnimating = true;
        
        // Transform Handle Module based on Gesture
        if (gesture === 'PALM') {
          handles[currentObj].className = "module-mesh mod-bar";
        } else if (gesture === 'POINT') {
          handles[currentObj].className = "module-mesh mod-button";
        } else if (gesture === 'ROTATE') {
          handles[currentObj].className = "module-mesh mod-knob";
        }

        setTimeout(() => {
          executeAction();
          setTimeout(() => {
            step = 4;
            updateText();
            isAnimating = false;
          }, 1800);
        }, 800);
      }
    }

    updateText();
  </script>
</body>
</html>
