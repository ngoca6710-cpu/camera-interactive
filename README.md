<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>DÙNG ĐƯỢC? — THE DOOR</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      user-select: none;
    }
    body {
      background-color: #0d0d0d;
      color: #f0f0f0;
      font-family: 'Courier New', Courier, monospace;
      overflow: hidden;
      height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
    }

    /* Start Overlay for Autoplay & Camera Permission */
    #start-screen {
      position: absolute;
      top: 0;
      left: 0;
      width: 100vw;
      height: 100vh;
      background: #0d0d0d;
      z-index: 100;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      text-align: center;
      padding: 20px;
    }

    .start-btn {
      margin-top: 20px;
      padding: 15px 30px;
      font-size: 1.2rem;
      background: #00ccff;
      color: #000;
      border: none;
      font-weight: bold;
      cursor: pointer;
      border-radius: 4px;
      transition: 0.2s;
    }
    .start-btn:hover {
      background: #00ff66;
    }

    #webcam-container {
      position: absolute;
      top: 20px;
      right: 20px;
      width: 160px;
      height: 120px;
      border: 1px solid #333;
      border-radius: 8px;
      overflow: hidden;
      z-index: 10;
      background: #000;
    }
    #webcam {
      width: 100%;
      height: 100%;
      object-fit: cover;
      transform: scaleX(-1);
    }

    #canvas-container {
      width: 100vw;
      height: 100vh;
      position: absolute;
      top: 0;
      left: 0;
      z-index: 1;
    }

    #ui-overlay {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      z-index: 5;
      pointer-events: none;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      padding: 40px;
      text-align: center;
    }

    .header-title {
      font-size: 1.2rem;
      letter-spacing: 2px;
      color: #888;
    }

    .prompt-box {
      background: rgba(0, 0, 0, 0.85);
      padding: 20px 30px;
      border-radius: 4px;
      border: 1px solid #333;
      backdrop-filter: blur(5px);
      max-width: 600px;
      margin: 0 auto;
      pointer-events: auto;
    }

    .prompt-text {
      font-size: 1.4rem;
      font-weight: bold;
      color: #fff;
      margin-bottom: 8px;
    }

    .sub-text {
      font-size: 0.9rem;
      color: #aaa;
    }

    .status-badge {
      display: inline-block;
      padding: 4px 12px;
      background: #222;
      border: 1px solid #444;
      border-radius: 12px;
      font-size: 0.8rem;
      color: #00ff66;
      margin-top: 10px;
    }

    /* Fallback Manual Controls */
    .controls-fallback {
      margin-top: 15px;
      display: flex;
      gap: 10px;
      justify-content: center;
    }

    .control-btn {
      padding: 6px 12px;
      background: #222;
      border: 1px solid #555;
      color: #fff;
      font-size: 0.8rem;
      cursor: pointer;
      border-radius: 4px;
    }
    .control-btn:hover {
      background: #444;
    }

    #stats-wall {
      display: none;
      background: rgba(0, 0, 0, 0.9);
      border: 1px solid #444;
      padding: 20px;
      max-width: 500px;
      margin: 0 auto;
      text-align: left;
    }

    .stat-number {
      font-size: 2.2rem;
      color: #ff3366;
      font-weight: bold;
    }
  </style>

  <!-- Three.js (3D Library) -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
  <!-- MediaPipe Hands CDN -->
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js" crossorigin="anonymous"></script>
</head>
<body>

  <!-- Screen Bắt Đầu -->
  <div id="start-screen">
    <h1 style="letter-spacing: 3px; margin-bottom: 10px;">DÙNG ĐƯỢC?</h1>
    <p style="color: #aaa; max-width: 400px;">CỬA MỞ BẰNG CỬ CHỈ BẢN THÂN. HÃY CHO PHÉP TRUY CẬP WEBCAM ĐỂ TRẢI NGHIỆM.</p>
    <button class="start-btn" onclick="initExperience()">BẮT ĐẦU TRẢI NGHIỆM</button>
  </div>

  <div id="webcam-container">
    <video id="webcam" autoplay playsinline muted></video>
  </div>

  <div id="canvas-container"></div>

  <div id="ui-overlay">
    <div class="header-title">DÙNG ĐƯỢC? / USABLE?</div>

    <div class="prompt-box" id="prompt-box">
      <div class="prompt-text" id="main-prompt">ĐÂY LÀ MỘT CÁNH CỬA.</div>
      <div class="sub-text" id="sub-prompt">Đưa tay trước webcam hoặc dùng phím thử nghiệm bên dưới:</div>
      <div class="status-badge" id="status-badge">Đang chờ nhận diện tay...</div>

      <!-- Nút thao tác bằng tay (Fallback dự phòng khi không có webcam) -->
      <div class="controls-fallback">
        <button class="control-btn" onclick="triggerGesture('PINCH')">Chụm Tay (Xoay)</button>
        <button class="control-btn" onclick="triggerGesture('PALM')">Xòe Tay (Thanh dài)</button>
        <button class="control-btn" onclick="triggerGesture('POINT')">Chỉ Ngón (Nút bấm)</button>
      </div>
    </div>

    <div id="stats-wall">
      <h2 style="margin-bottom: 15px; color: #fff;">BỨC TƯỜNG DÙNG ĐƯỢC</h2>
      <p>HÔM NAY:</p>
      <div class="stat-number">183 NGƯỜI</div>
      <div class="stat-number" style="color: #00ff66;">183 CỬ CHỈ</div>
      <div class="stat-number" style="color: #00ccff;">27 CÁCH MỞ CỬA MỚI</div>
      <p style="margin-top: 15px; font-size: 0.8rem; color: #888;">"THAY ĐỔI CÁCH BẠN DÙNG. THAY ĐỔI THIẾT KẾ."</p>
    </div>

    <div style="font-size: 0.8rem; color: #555;">Mô hình tương tác DÙNG ĐƯỢC? (Usable?)</div>
  </div>

  <script>
    let currentStage = 1;
    let gestureState = "NONE";
    let isDoorOpen = false;
    let stageTimer = null;

    const mainPrompt = document.getElementById('main-prompt');
    const subPrompt = document.getElementById('sub-prompt');
    const statusBadge = document.getElementById('status-badge');
    const statsWall = document.getElementById('stats-wall');
    const promptBox = document.getElementById('prompt-box');

    // --- 1. DỰNG CẢNH THREE.JS (3D) ---
    const container = document.getElementById('canvas-container');
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x0d0d0d);

    const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.position.set(0, 0, 7);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    container.appendChild(renderer.domElement);

    const ambientLight = new THREE.AmbientLight(0xffffff, 0.7);
    scene.add(ambientLight);

    const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
    dirLight.position.set(5, 10, 7);
    scene.add(dirLight);

    // Cấu trúc Khung & Cửa
    const doorGroup = new THREE.Group();
    scene.add(doorGroup);

    const frameGeo = new THREE.BoxGeometry(2.6, 4.2, 0.1);
    const frameMat = new THREE.MeshStandardMaterial({ color: 0x222222 });
    const frame = new THREE.Mesh(frameGeo, frameMat);
    doorGroup.add(frame);

    const doorPivot = new THREE.Group();
    doorPivot.position.set(-1.2, 0, 0.05);
    doorGroup.add(doorPivot);

    const doorGeo = new THREE.BoxGeometry(2.4, 4.0, 0.08);
    const doorMat = new THREE.MeshStandardMaterial({ color: 0x333333, roughness: 0.4 });
    const doorMesh = new THREE.Mesh(doorGeo, doorMat);
    doorMesh.position.set(1.2, 0, 0);
    doorPivot.add(doorMesh);

    // Tay cầm biến đổi (Handle)
    const handleGroup = new THREE.Group();
    handleGroup.position.set(2.0, 0, 0.08);
    doorPivot.add(handleGroup);

    const knobGeo = new THREE.CylinderGeometry(0.12, 0.12, 0.1, 32);
    const buttonGeo = new THREE.BoxGeometry(0.4, 0.4, 0.1);
    const barGeo = new THREE.BoxGeometry(0.1, 3.0, 0.1);

    const handleMat = new THREE.MeshStandardMaterial({ color: 0x00ccff, metalness: 0.8, roughness: 0.2 });
    let currentHandleMesh = new THREE.Mesh(knobGeo, handleMat);
    currentHandleMesh.rotation.x = Math.PI / 2;
    handleGroup.add(currentHandleMesh);

    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });

    // --- 2. CÁC GIAI ĐOẠN TRẢI NGHIỆM ---
    function updateStageUI() {
      if (currentStage === 1) {
        mainPrompt.innerText = "01 — CÁCH MỞ QUEN THUỘC";
        subPrompt.innerText = "Tạo cử chỉ xoay/chụm tay để mở cửa.";
      } else if (currentStage === 2) {
        mainPrompt.innerText = "02 — NHƯNG NẾU BẠN KHÔNG THỂ XOAY CỔ TAY?";
        subPrompt.innerText = "Tay cầm đang tách khỏi cửa thành các mô-đun...";
      } else if (currentStage === 3) {
        mainPrompt.innerText = "03 — TẠO CÁCH MỞ MỚI BẰNG CỬ CHỈ CỦA BẠN";
        subPrompt.innerText = "Thay đổi dáng tay để biến đổi hình dạng thiết kế tay cầm.";
      } else if (currentStage === 4) {
        promptBox.style.display = "none";
        statsWall.style.display = "block";
      }
    }

    function setHandleType(type) {
      handleGroup.remove(currentHandleMesh);
      if (type === 'KNOB') {
        currentHandleMesh = new THREE.Mesh(knobGeo, handleMat);
        currentHandleMesh.rotation.x = Math.PI / 2;
      } else if (type === 'BUTTON') {
        currentHandleMesh = new THREE.Mesh(buttonGeo, handleMat);
      } else if (type === 'BAR') {
        currentHandleMesh = new THREE.Mesh(barGeo, handleMat);
      }
      handleGroup.add(currentHandleMesh);
    }

    function openDoor() {
      if (isDoorOpen) return;
      isDoorOpen = true;
      let angle = 0;
      const interval = setInterval(() => {
        angle += 0.05;
        doorPivot.rotation.y = -angle;
        if (angle >= Math.PI / 2.5) {
          clearInterval(interval);
          setTimeout(closeDoor, 1500);
        }
      }, 16);
    }

    function closeDoor() {
      let angle = doorPivot.rotation.y;
      const interval = setInterval(() => {
        angle += 0.05;
        doorPivot.rotation.y = angle;
        if (angle >= 0) {
          doorPivot.rotation.y = 0;
          clearInterval(interval);
          isDoorOpen = false;
          advanceStage();
        }
      }, 16);
    }

    function advanceStage() {
      if (currentStage === 1) {
        currentStage = 2;
        updateStageUI();
        setTimeout(() => {
          currentStage = 3;
          updateStageUI();
        }, 3000);
      }
    }

    // Nút kích hoạt giả lập cử chỉ (Fallback Mode)
    function triggerGesture(type) {
      if (type === 'PINCH') {
        processGestureInput('PINCH / ROTATE');
      } else if (type === 'PALM') {
        processGestureInput('OPEN PALM');
      } else if (type === 'POINT') {
        processGestureInput('POINTING');
      }
    }

    function processGestureInput(gesture) {
      statusBadge.innerText = `Cử chỉ: ${gesture}`;
      
      if (currentStage === 1 && (gesture === 'PINCH / ROTATE' || gesture === 'OPEN PALM')) {
        openDoor();
      } else if (currentStage === 3) {
        if (gesture === 'OPEN PALM') setHandleType('BAR');
        if (gesture === 'POINTING') setHandleType('BUTTON');
        if (gesture === 'PINCH / ROTATE') setHandleType('KNOB');

        if (!isDoorOpen) {
          clearTimeout(stageTimer);
          stageTimer = setTimeout(() => {
            openDoor();
            setTimeout(() => {
              currentStage = 4;
              updateStageUI();
            }, 2000);
          }, 2000);
        }
      }
    }

    // --- 3. KHỞI TẠO MEDIAPIPE WEBCAM ---
    function initExperience() {
      document.getElementById('start-screen').style.display = 'none';

      const videoElement = document.getElementById('webcam');

      const hands = new Hands({
        locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`
      });

      hands.setOptions({
        maxNumHands: 1,
        modelComplexity: 1,
        minDetectionConfidence: 0.5,
        minTrackingConfidence: 0.5
      });

      hands.onResults((results) => {
        if (results.multiHandLandmarks && results.multiHandLandmarks.length > 0) {
          const landmarks = results.multiHandLandmarks[0];
          const thumb = landmarks[4];
          const index = landmarks[8];
          const wrist = landmarks[0];
          const middle = landmarks[12];

          const pinchDist = Math.hypot(thumb.x - index.x, thumb.y - index.y);
          const openDist = Math.hypot(wrist.x - middle.x, wrist.y - middle.y);

          let detected = "HAND DETECTED";
          if (pinchDist < 0.08) detected = "PINCH / ROTATE";
          else if (openDist > 0.35) detected = "OPEN PALM";
          else if (index.y < landmarks[6].y) detected = "POINTING";

          processGestureInput(detected);
        } else {
          statusBadge.innerText = "Đang quét bàn tay...";
        }
      });

      const cameraUtils = new Camera(videoElement, {
        onFrame: async () => {
          await hands.send({ image: videoElement });
        },
        width: 640,
        height: 480
      });

      cameraUtils.start().catch(err => {
        console.warn("Không thể mở Webcam, bạn có thể dùng các nút bên dưới:", err);
        statusBadge.innerText = "Webcam không khả dụng - Dùng nút bấm thử nghiệm";
      });

      updateStageUI();
    }

    // Animation Loop
    function animate() {
      requestAnimationFrame(animate);
      if (currentStage === 2) {
        handleGroup.rotation.y += 0.05;
        handleGroup.rotation.x += 0.02;
      } else {
        handleGroup.rotation.set(0, 0, 0);
        if (currentHandleMesh.geometry.type === "CylinderGeometry") {
          currentHandleMesh.rotation.x = Math.PI / 2;
        }
      }
      renderer.render(scene, camera);
    }
    animate();
  </script>
</body>
</html>
