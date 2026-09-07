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
      width: 100vw;
    }

    #canvas-container {
      width: 100%;
      height: 100%;
      position: absolute;
      top: 0;
      left: 0;
      z-index: 1;
    }

    #webcam-container {
      position: absolute;
      top: 20px;
      right: 20px;
      width: 140px;
      height: 105px;
      border: 1px solid #444;
      border-radius: 6px;
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
      padding: 30px;
      text-align: center;
    }

    .header-title {
      font-size: 1.1rem;
      letter-spacing: 2px;
      color: #888;
    }

    .prompt-box {
      background: rgba(0, 0, 0, 0.85);
      padding: 20px 25px;
      border-radius: 6px;
      border: 1px solid #333;
      backdrop-filter: blur(5px);
      max-width: 550px;
      margin: 0 auto;
      pointer-events: auto;
    }

    .prompt-text {
      font-size: 1.3rem;
      font-weight: bold;
      color: #fff;
      margin-bottom: 8px;
    }

    .sub-text {
      font-size: 0.85rem;
      color: #aaa;
    }

    .status-badge {
      display: inline-block;
      padding: 4px 12px;
      background: #111;
      border: 1px solid #00ff66;
      border-radius: 12px;
      font-size: 0.75rem;
      color: #00ff66;
      margin-top: 10px;
    }

    .controls-fallback {
      margin-top: 15px;
      display: flex;
      gap: 8px;
      justify-content: center;
    }

    .control-btn {
      padding: 8px 12px;
      background: #222;
      border: 1px solid #555;
      color: #fff;
      font-size: 0.75rem;
      cursor: pointer;
      border-radius: 4px;
    }
    .control-btn:hover {
      background: #00ccff;
      color: #000;
    }

    #stats-wall {
      display: none;
      background: rgba(0, 0, 0, 0.9);
      border: 1px solid #444;
      padding: 20px;
      max-width: 450px;
      margin: 0 auto;
      text-align: left;
    }

    .stat-number {
      font-size: 2rem;
      color: #ff3366;
      font-weight: bold;
    }
  </style>

  <!-- Nạp Three.js từ CDN chính thức -->
  <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/build/three.min.js"></script>
</head>
<body>

  <div id="webcam-container">
    <video id="webcam" autoplay playsinline muted></video>
  </div>

  <div id="canvas-container"></div>

  <div id="ui-overlay">
    <div class="header-title">DÙNG ĐƯỢC? / USABLE?</div>

    <div class="prompt-box" id="prompt-box">
      <div class="prompt-text" id="main-prompt">01 — ĐÂY LÀ MỘT CÁNH CỬA</div>
      <div class="sub-text" id="sub-prompt">Bấm các phím cử chỉ dưới đây hoặc bật Webcam để thử nghiệm:</div>
      <div class="status-badge" id="status-badge">Trạng thái: Đã sẵn sàng</div>

      <div class="controls-fallback">
        <button class="control-btn" onclick="handleGesture('PINCH')">1. Chụm tay (Xoay)</button>
        <button class="control-btn" onclick="handleGesture('PALM')">2. Xòe tay (Thanh dài)</button>
        <button class="control-btn" onclick="handleGesture('POINT')">3. Chỉ ngón (Nút bấm)</button>
      </div>
    </div>

    <div id="stats-wall">
      <h2 style="margin-bottom: 12px; color: #fff;">BỨC TƯỜNG DÙNG ĐƯỢC</h2>
      <p>HÔM NAY:</p>
      <div class="stat-number">183 NGƯỜI</div>
      <div class="stat-number" style="color: #00ff66;">183 CỬ CHỈ</div>
      <div class="stat-number" style="color: #00ccff;">27 CÁCH MỞ CỬA MỚI</div>
      <p style="margin-top: 12px; font-size: 0.8rem; color: #888;">"THAY ĐỔI CÁCH BẠN DÙNG. THAY ĐỔI THIẾT KẾ."</p>
    </div>

    <div style="font-size: 0.75rem; color: #555;">DỰ ÁN NGHỆ THUẬT TƯƠNG TÁC DÙNG ĐƯỢC?</div>
  </div>

  <script>
    let currentStage = 1;
    let isDoorOpen = false;
    let stageTimer = null;

    const mainPrompt = document.getElementById('main-prompt');
    const subPrompt = document.getElementById('sub-prompt');
    const statusBadge = document.getElementById('status-badge');
    const statsWall = document.getElementById('stats-wall');
    const promptBox = document.getElementById('prompt-box');

    // --- 1. DỰNG MÔ HÌNH 3D VỚI THREE.JS ---
    const container = document.getElementById('canvas-container');
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x0d0d0d);

    const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.position.set(0, 0, 7);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    container.appendChild(renderer.domElement);

    const ambientLight = new THREE.AmbientLight(0xffffff, 0.7);
    scene.add(ambientLight);

    const dirLight = new THREE.DirectionalLight(0xffffff, 0.9);
    dirLight.position.set(5, 10, 7);
    scene.add(dirLight);

    // Khung & Cánh cửa
    const doorGroup = new THREE.Group();
    scene.add(doorGroup);

    const frame = new THREE.Mesh(
      new THREE.BoxGeometry(2.6, 4.2, 0.1),
      new THREE.MeshStandardMaterial({ color: 0x222222 })
    );
    doorGroup.add(frame);

    const doorPivot = new THREE.Group();
    doorPivot.position.set(-1.2, 0, 0.05);
    doorGroup.add(doorPivot);

    const doorMesh = new THREE.Mesh(
      new THREE.BoxGeometry(2.4, 4.0, 0.08),
      new THREE.MeshStandardMaterial({ color: 0x333333, roughness: 0.4 })
    );
    doorMesh.position.set(1.2, 0, 0);
    doorPivot.add(doorMesh);

    // Tay cầm mô-đun
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

    // --- 2. CÁC THAO TÁC KỊCH BẢN ---
    function updateStageUI() {
      if (currentStage === 1) {
        mainPrompt.innerText = "01 — CÁCH MỞ QUEN THUỘC";
        subPrompt.innerText = "Bấm '1. Chụm tay' để thực hiện thao tác xoay mở cửa.";
      } else if (currentStage === 2) {
        mainPrompt.innerText = "02 — NHƯNG NẾU BẠN KHÔNG THỂ XOAY CỔ TAY?";
        subPrompt.innerText = "Cơ cấu tay cầm đang biến đổi thành các mô-đun...";
      } else if (currentStage === 3) {
        mainPrompt.innerText = "03 — TẠO CÁCH MỞ MỚI BẰNG CỬ CHỈ CỦA BẠN";
        subPrompt.innerText = "Chọn các dạng cử chỉ phía dưới để tạo hình dạng tay cầm mới.";
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

    function handleGesture(type) {
      if (currentStage === 1 && type === 'PINCH') {
        statusBadge.innerText = "Thực hiện: Xoay cổ tay -> Mở cửa";
        openDoor();
      } else if (currentStage === 3) {
        if (type === 'PALM') {
          setHandleType('BAR');
          statusBadge.innerText = "Biến hình: Thanh trượt dài";
        } else if (type === 'POINT') {
          setHandleType('BUTTON');
          statusBadge.innerText = "Biến hình: Nút bấm lớn";
        } else if (type === 'PINCH') {
          setHandleType('KNOB');
          statusBadge.innerText = "Biến hình: Tay nắm tròn";
        }

        if (!isDoorOpen) {
          clearTimeout(stageTimer);
          stageTimer = setTimeout(() => {
            openDoor();
            setTimeout(() => {
              currentStage = 4;
              updateStageUI();
            }, 2000);
          }, 1500);
        }
      }
    }

    // --- 3. KÍCH HOẠT CAMERA MẶC ĐỊNH TRÌNH DUYỆT ---
    async function startWebcam() {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        document.getElementById('webcam').srcObject = stream;
      } catch (e) {
        statusBadge.innerText = "Không dùng Camera - Bạn có thể dùng nút bấm bên dưới";
      }
    }

    // Khởi chạy
    startWebcam();
    updateStageUI();

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
