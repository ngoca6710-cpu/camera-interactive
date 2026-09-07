<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>DÙNG ĐƯỢC? — AI HAND GESTURE</title>
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

    /* Thanh chọn đồ vật */
    #nav-bar {
      display: flex;
      gap: 10px;
      z-index: 10;
      background: rgba(20, 20, 20, 0.9);
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
      padding: 4px 10px;
      border-radius: 12px;
      transition: all 0.3s;
    }

    .nav-btn.active {
      color: #00ff66;
      background: #222;
      font-weight: bold;
    }

    /* Ô hiển thị Camera thu nhỏ */
    #cam-preview {
      position: absolute;
      top: 15px;
      right: 15px;
      width: 120px;
      height: 90px;
      border: 1px solid #00ff66;
      border-radius: 8px;
      overflow: hidden;
      background: #000;
      z-index: 20;
    }
    #webcam {
      width: 100%;
      height: 100%;
      object-fit: cover;
      transform: scaleX(-1);
    }

    /* Viewport hiển thị 3D */
    #viewport {
      perspective: 1000px;
      width: 260px;
      height: 300px;
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

    /* Đồ vật 01: CỬA */
    #door-frame { width: 180px; height: 270px; border: 4px solid #333; background: #111; position: relative; }
    #door-leaf {
      width: 100%; height: 100%; background: #222; border: 1px solid #444;
      transform-origin: left; transition: transform 1.2s cubic-bezier(0.4, 0, 0.2, 1);
      display: flex; align-items: center; justify-content: flex-end; padding-right: 15px;
    }

    /* Đồ vật 02: VÒI NƯỚC */
    #faucet-body { width: 40px; height: 130px; background: #333; border-radius: 20px 20px 0 0; position: relative; display: flex; flex-direction: column; align-items: center; }
    #faucet-spout { width: 70px; height: 20px; background: #333; position: absolute; top: 20px; right: -50px; border-radius: 0 10px 10px 0; }
    #water-stream { width: 12px; height: 0px; background: #00ccff; position: absolute; top: 40px; right: -46px; transition: height 0.5s; opacity: 0.8; }

    /* Đồ vật 03: CÔNG TẮC ĐÈN */
    #switch-plate { width: 150px; height: 150px; background: #222; border: 2px solid #444; border-radius: 12px; display: flex; align-items: center; justify-content: center; position: relative; }
    #light-bulb { width: 30px; height: 30px; border-radius: 50%; background: #333; position: absolute; top: -45px; transition: all 0.3s; }

    /* MÔ-ĐUN TAY CẦM BIẾN HÌNH */
    .module-mesh {
      background: #00ccff;
      border-radius: 4px;
      transition: all 0.6s cubic-bezier(0.68, -0.55, 0.265, 1.55);
      box-shadow: 0 0 12px rgba(0, 204, 255, 0.6);
    }

    .mod-knob { width: 22px; height: 22px; border-radius: 50% !important; transform: translate(0, 0); }
    .mod-bar { width: 10px; height: 160px; border-radius: 5px !important; transform: translate(0, 0); }
    .mod-button { width: 50px; height: 50px; border-radius: 10px !important; transform: translate(0, 0); }
    .mod-detached { transform: translate(-40px, -30px) scale(0.6) rotate(45deg); opacity: 0.7; }

    /* Bảng trạng thái & Bắt cử chỉ */
    #ui-card {
      background: rgba(20, 20, 20, 0.95);
      border: 1px solid #333;
      padding: 16px;
      border-radius: 10px;
      text-align: center;
      max-width: 420px;
      width: 100%;
    }

    .title { font-size: 0.95rem; font-weight: bold; color: #00ccff; margin-bottom: 6px; }
    .desc { font-size: 0.8rem; color: #aaa; margin-bottom: 10px; min-height: 32px; }

    #gesture-badge {
      display: inline-block;
      padding: 6px 16px;
      background: #111;
      border: 1px solid #00ff66;
      color: #00ff66;
      border-radius: 20px;
      font-size: 0.85rem;
      font-weight: bold;
    }

    #wall-of-ways { display: none; text-align: left; }
    .stat-val { font-size: 1.5rem; font-weight: bold; color: #ff3366; }
  </style>

  <!-- Nạp AI MediaPipe nhận diện bàn tay -->
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js" crossorigin="anonymous"></script>
</head>
<body>

  <!-- Ô thu nhỏ Camera -->
  <div id="cam-preview">
    <video id="webcam" autoplay playsinline muted></video>
  </div>

  <!-- Thanh chọn Đồ vật -->
  <div id="nav-bar">
    <button class="nav-btn active" onclick="switchObject('DOOR')">01. CỬA</button>
    <button class="nav-btn" onclick="switchObject('FAUCET')">02. VÒI NƯỚC</button>
    <button class="nav-btn" onclick="switchObject('SWITCH')">03. CÔNG TẮC</button>
  </div>

  <!-- Viewport 3D -->
  <div id="viewport">
    <!-- CỬA -->
    <div id="obj-door" class="object-container">
      <div id="door-frame">
        <div id="door-leaf">
          <div id="door-handle" class="module-mesh mod-knob"></div>
        </div>
      </div>
    </div>

    <!-- VÒI NƯỚC -->
    <div id="obj-faucet" class="object-container" style="opacity: 0;">
      <div id="faucet-body">
        <div id="faucet-spout">
          <div id="water-stream"></div>
        </div>
        <div id="faucet-handle" class="module-mesh mod-knob" style="margin-top: -15px;"></div>
      </div>
    </div>

    <!-- CÔNG TẮC -->
    <div id="obj-switch" class="object-container" style="opacity: 0;">
      <div id="light-bulb"></div>
      <div id="switch-plate">
        <div id="switch-handle" class="module-mesh mod-knob"></div>
      </div>
    </div>
  </div>

  <!-- Khung kịch bản -->
  <div id="ui-card">
    <div id="interactive-panel">
      <div class="title" id="p-title">01 — CÁCH DÙNG QUEN THUỘC</div>
      <div class="desc" id="p-desc">Hãy chụm các ngón tay lại (Cử chỉ vặn/xoay) để vận hành.</div>
      <div id="gesture-badge">Đang quét bàn tay...</div>
    </div>

    <div id="wall-of-ways">
      <div style="color:#fff; font-weight:bold; margin-bottom:6px;">BỨC TƯỜNG DÙNG ĐƯỢC</div>
      <div class="stat-val">183 NGƯỜI THAM GIA</div>
      <div class="stat-val" style="color:#00ff66;">183 CỬ CHỈ CỬ ĐỘNG</div>
      <div class="stat-val" style="color:#00ccff;">42 CÁCH MỞ MỚI</div>
      <p style="margin-top:6px; font-size:0.7rem; color:#aaa;">"THAY ĐỔI CÁCH BẠN DÙNG. THAY ĐỔI THIẾT KẾ."</p>
    </div>
  </div>

  <script>
    let currentObj = 'DOOR';
    let step = 1;
    let isAnimating = false;

    const pTitle = document.getElementById('p-title');
    const pDesc = document.getElementById('p-desc');
    const gestureBadge = document.getElementById('gesture-badge');
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

      document.querySelectorAll('.nav-btn').forEach((btn, idx) => {
        btn.classList.toggle('active', (idx === 0 && objType==='DOOR') || (idx === 1 && objType==='FAUCET') || (idx === 2 && objType==='SWITCH'));
      });

      document.getElementById('obj-door').style.opacity = objType === 'DOOR' ? '1' : '0';
      document.getElementById('obj-faucet').style.opacity = objType === 'FAUCET' ? '1' : '0';
      document.getElementById('obj-switch').style.opacity = objType === 'SWITCH' ? '1' : '0';

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
        pDesc.innerText = "Chụm tay lại (Bóp/Xoay) để vận hành theo cách truyền thống.";
      } else if (step === 2) {
        pTitle.innerText = "02 — NHƯNG NẾU BẠN KHÔNG THỂ XOAY CỔ TAY?";
        pDesc.innerText = "Cơ cấu đang tự động rã thành các mô-đun biến hình...";
      } else if (step === 3) {
        pTitle.innerText = "03 — TẠO THIẾT KẾ BẰNG BÀN TAY BẠN";
        pDesc.innerText = "Xòe rộng bàn tay (Thanh dài) hoặc Chỉ 1 ngón tay (Nút bấm).";
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

    function processAIHandGesture(detectedGesture) {
      if (isAnimating) return;

      gestureBadge.innerText = `Cử chỉ: ${detectedGesture}`;

      if (step === 1 && detectedGesture === 'PINCH / CHỤM TAY') {
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
        if (detectedGesture === 'OPEN PALM / XÒE TAY') {
          handles[currentObj].className = "module-mesh mod-bar";
        } else if (detectedGesture === 'POINTING / CHỈ NGÓN') {
          handles[currentObj].className = "module-mesh mod-button";
        } else if (detectedGesture === 'PINCH / CHỤM TAY') {
          handles[currentObj].className = "module-mesh mod-knob";
        } else {
          return;
        }

        isAnimating = true;
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

    // --- KHỞI TẠO WEBCAM VÀ AI MEDIAPIPE ---
    const videoElement = document.getElementById('webcam');

    const hands = new Hands({
      locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`
    });

    hands.setOptions({
      maxNumHands: 1,
      modelComplexity: 1,
      minDetectionConfidence: 0.6,
      minTrackingConfidence: 0.6
    });

    hands.onResults((results) => {
      if (results.multiHandLandmarks && results.multiHandLandmarks.length > 0) {
        const landmarks = results.multiHandLandmarks[0];
        
        const thumbTip = landmarks[4];
        const indexTip = landmarks[8];
        const wrist = landmarks[0];
        const middleTip = landmarks[12];

        // Khoảng cách nhận diện các dáng tay
        const pinchDist = Math.hypot(thumbTip.x - indexTip.x, thumbTip.y - indexTip.y);
        const openDist = Math.hypot(wrist.x - middleTip.x, wrist.y - middleTip.y);

        let detected = "ĐANG TÌM CỬ CHỈ";
        if (pinchDist < 0.07) {
          detected = "PINCH / CHỤM TAY";
        } else if (openDist > 0.38) {
          detected = "OPEN PALM / XÒE TAY";
        } else if (indexTip.y < landmarks[6].y) {
          detected = "POINTING / CHỈ NGÓN";
        }

        processAIHandGesture(detected);
      } else {
        gestureBadge.innerText = "Đưa tay trước webcam...";
      }
    });

    const cameraUtils = new Camera(videoElement, {
      onFrame: async () => {
        await hands.send({ image: videoElement });
      },
      width: 320,
      height: 240
    });

    cameraUtils.start().catch(() => {
      gestureBadge.innerText = "Không thể mở Webcam";
    });

    updateText();
  </script>
</body>
</html>
