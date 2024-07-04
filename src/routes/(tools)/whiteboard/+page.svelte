<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Whiteboard</title>
  <style>
    .inner-card {
      height: 600px;
      position: relative;
    }

    #whiteboard {
      width: 100%;
      height: 100%;
      border: 1px solid #ccc;
    }

    .nav-bar {
      width: 100%;
      position: relative;
    }

    .tool-btn {
      border: none;
      padding: 8px 16px;
      margin: 4px;
      cursor: pointer;
      border-radius: 4px;
      position: relative;
    }

    .tool-btn img {
      width: 24px;
      height: 24px;
    }

    .tool-btn:hover {
      background-color: #e0e0e0;
    }

    .dropdown {
      position: absolute;
      top: calc(100% + 8px);
      left: 0;
      z-index: 1000;
      display: none;
      background-color: #fff;
      box-shadow: 0 1px 2px rgba(0, 0, 0, 0.15);
      border-radius: 4px;
      width: max-content;
    }

    .dropdown.active {
      display: block;
    }

    .dropdown-arrow {
      position: absolute;
      top: 100%;
      left: 50%;
      transform: translateX(-50%);
      width: 0;
      height: 0;
      border-style: solid;
      border-width: 6px 6px 0 6px;
      border-color: #f0f0f0 transparent transparent transparent;
    }

    .dropdown-item {
      padding: 8px 16px;
      cursor: pointer;
    }

    .dropdown-item:hover {
      background-color: #f0f0f0;
    }

    .color-palette {
      display: flex;
      padding: 8px;
    }

    .color-swatch {
      width: 24px;
      height: 24px;
      margin: 4px;
      border-radius: 4px;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <div class="card items-center mx-auto max-w-screen-xl lg:grid overflow-hidden rounded-lg">
    <!-- Navigation bar -->
    <div class="nav-bar bg-gray-200 p-2 flex items-center">
      <button id="pencil" class="tool-btn"><img src="/src/images/pencil.png" alt=""/></button>
      <div class="relative">
        <button id="eraser" class="tool-btn"><img src="/src/images/eraser.png" alt=""/><span class="dropdown-arrow"></span></button>
        <div id="eraser-width-dropdown" class="dropdown">
          <div class="dropdown-item" data-width="5">5px</div>
          <div class="dropdown-item" data-width="10">10px</div>
          <div class="dropdown-item" data-width="15">15px</div>
          <div class="dropdown-item" data-width="20">20px</div>
        </div>
      </div>
      <div class="relative">
        <button id="fill" class="tool-btn"><img src="/src/images/paint-bucket.png" alt=""/><span class="dropdown-arrow"></span></button>
        <div id="color-palette-dropdown" class="dropdown">
          <div class="color-palette">
            <div class="color-swatch" data-color="#FF0000" style="background-color: #FF0000;"></div>
            <div class="color-swatch" data-color="#00FF00" style="background-color: #00FF00;"></div>
            <div class="color-swatch" data-color="#0000FF" style="background-color: #0000FF;"></div>
            <div class="color-swatch" data-color="#FFFF00" style="background-color: #FFFF00;"></div>
            <div class="color-swatch" data-color="#FF00FF" style="background-color: #FF00FF;"></div>
            <div class="color-swatch" data-color="#00FFFF" style="background-color: #00FFFF;"></div>
            <div class="color-swatch" data-color="#000000" style="background-color: #000000;"></div>
            <div class="color-swatch" data-color="#FFFFFF" style="background-color: #FFFFFF;"></div>
          </div>
        </div>
      </div>
      <div class="relative">
        <button id="shapes" class="tool-btn"><img src="/src/images/shapes.png" alt=""/><span class="dropdown-arrow"></span></button>
        <div id="shapes-dropdown" class="dropdown">
          <div class="dropdown-item" data-shape="rectangle">Rectangle</div>
          <div class="dropdown-item" data-shape="triangle">Triangle</div>
          <div class="dropdown-item" data-shape="circle">Circle</div>
          <div class="dropdown-item" data-shape="square">Square</div>
          <div class="dropdown-item" data-shape="line">Line</div>
        </div>
      </div>
      <button id="undo" class="tool-btn"><img src="/src/images/undo.png" alt=""/></button>
      <button id="redo" class="tool-btn"><img src="/src/images/redo.png" alt=""/></button>
      <button id="clear" class="tool-btn"><img src="/src/images/clear.png" alt="Clear Canvas"/></button>
    </div>
    
    <!-- Whiteboard area -->
    <div class="inner-card w-full bg-white">
      <canvas id="whiteboard"></canvas>
    </div>
  </div>

  <script>
    document.addEventListener('DOMContentLoaded', function() {
      const canvas = document.getElementById('whiteboard');
      const ctx = canvas.getContext('2d');
      let isDrawing = false;
      let lastX = 0;
      let lastY = 0;
      let eraserSize = 10;
      let drawingHistory = [];
      let historyIndex = -1;
      let fillColor = [0, 0, 0, 255]; // Default fill color is black
      let pencilActive = true;
      let eraserActive = false;
      let shapeActive = false;
      let currentShape = '';
      let shapeStartX = 0;
      let shapeStartY = 0;
      let shapeEndX = 0;
      let shapeEndY = 0;
      canvas.width = canvas.offsetWidth;
      canvas.height = canvas.offsetHeight;
      function startDrawing(e) {
        isDrawing = true;
        [lastX, lastY] = [e.offsetX, e.offsetY];
        if (shapeActive) {
          shapeStartX = e.offsetX;
          shapeStartY = e.offsetY;
        }
      }
      function draw(e) {
        if (!isDrawing) return;
        ctx.lineWidth = 2;
        ctx.lineCap = 'round';
        if (pencilActive) {
          ctx.strokeStyle = '#000'; 
          ctx.beginPath();
          ctx.moveTo(lastX, lastY);
          ctx.lineTo(e.offsetX, e.offsetY);
          ctx.stroke();
          [lastX, lastY] = [e.offsetX, e.offsetY];
        } else if (eraserActive) {
          ctx.strokeStyle = '#fff'; 
          ctx.lineWidth = eraserSize; 
          ctx.beginPath();
          ctx.moveTo(lastX, lastY);
          ctx.lineTo(e.offsetX, e.offsetY);
          ctx.stroke();
          [lastX, lastY] = [e.offsetX, e.offsetY];
        } else if (shapeActive) {
          shapeEndX = e.offsetX;
          shapeEndY = e.offsetY;
        }
      }
      function stopDrawing() {
        if (!isDrawing) return;
        isDrawing = false;
        if (shapeActive) {
          drawShape(shapeEndX, shapeEndY);
          shapeActive = false;
        }
        if (!eraserActive) {
          saveDrawingAction();
        }
      }
      function saveDrawingAction() {
        if (historyIndex < drawingHistory.length - 1) {
          drawingHistory = drawingHistory.slice(0, historyIndex + 1);
        }
        drawingHistory.push(ctx.getImageData(0, 0, canvas.width, canvas.height));
        historyIndex++;
      }
      function undoDrawing() {
        if (historyIndex > 0) {
          historyIndex--;
          ctx.putImageData(drawingHistory[historyIndex], 0, 0);
        }
      }
      function redoDrawing() {
        if (historyIndex < drawingHistory.length - 1) {
          historyIndex++;
          ctx.putImageData(drawingHistory[historyIndex], 0, 0);
        }
      }
      function clearCanvas() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        saveDrawingAction();
      }
      function floodFill(x, y, fillColor) {
        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const data = imageData.data;
        const targetColor = getColorAtPixel(data, x, y);
        if (colorsMatch(targetColor, fillColor)) return;
        const stack = [[x, y]];
        while (stack.length) {
          const [currentX, currentY] = stack.pop();
          const currentPos = (currentY * canvas.width + currentX) * 4;
          if (!colorsMatch(getColorAtPixel(data, currentX, currentY), targetColor)) continue;
          setColorAtPixel(data, currentX, currentY, fillColor);
          if (currentX > 0) stack.push([currentX - 1, currentY]);
          if (currentX < canvas.width - 1) stack.push([currentX + 1, currentY]);
          if (currentY > 0) stack.push([currentX, currentY - 1]);
          if (currentY < canvas.height - 1) stack.push([currentX, currentY + 1]);
        }
        ctx.putImageData(imageData, 0, 0);
      }
      function getColorAtPixel(data, x, y) {
        const pos = (y * canvas.width + x) * 4;
        return [data[pos], data[pos + 1], data[pos + 2], data[pos + 3]];
      }
      function setColorAtPixel(data, x, y, color) {
        const pos = (y * canvas.width + x) * 4;
        data[pos] = color[0];
        data[pos + 1] = color[1];
        data[pos + 2] = color[2];
        data[pos + 3] = color[3];
      }
      function colorsMatch(color1, color2) {
        return color1[0] === color2[0] && color1[1] === color2[1] && color1[2] === color2[2] && color1[3] === color2[3];
      }
      function setTool(tool) {
        pencilActive = tool === 'pencil';
        eraserActive = tool === 'eraser';
        shapeActive = tool === 'shape';
      }
      function drawShape(x, y) {
        if (currentShape === 'rectangle') {
          const width = x - shapeStartX;
          const height = y - shapeStartY;
          ctx.strokeStyle = '#000'; 
          ctx.strokeRect(shapeStartX, shapeStartY, width, height);
        } else if (currentShape === 'circle') {
          const radius = Math.sqrt(Math.pow(x - shapeStartX, 2) + Math.pow(y - shapeStartY, 2));
          ctx.strokeStyle = '#000'; 
          ctx.beginPath();
          ctx.arc(shapeStartX, shapeStartY, radius, 0, Math.PI * 2);
          ctx.stroke();
        }
      }
      document.getElementById('pencil').addEventListener('click', function() {
        setTool('pencil');
      });
      document.getElementById('eraser').addEventListener('click', function() {
        setTool('eraser');
      });
      document.getElementById('fill').addEventListener('click', function() {
        setTool('fill');
      });
      document.getElementById('shapes').addEventListener('click', function() {
        setTool('shape');
      });
      document.getElementById('undo').addEventListener('click', undoDrawing);
      document.getElementById('redo').addEventListener('click', redoDrawing);
      document.getElementById('clear').addEventListener('click', clearCanvas);
      canvas.addEventListener('mousedown', startDrawing);
      canvas.addEventListener('mousemove', draw);
      canvas.addEventListener('mouseup', stopDrawing);
      canvas.addEventListener('mouseout', stopDrawing);
      const eraserWidthDropdown = document.getElementById('eraser-width-dropdown');
      document.getElementById('eraser').addEventListener('click', function() {
        eraserWidthDropdown.classList.toggle('active');
      });
      document.querySelectorAll('#eraser-width-dropdown .dropdown-item').forEach(item => {
        item.addEventListener('click', function() {
          eraserSize = parseInt(this.getAttribute('data-width'));
          eraserWidthDropdown.classList.remove('active');
        });
      });
      const colorPaletteDropdown = document.getElementById('color-palette-dropdown');
      document.getElementById('fill').addEventListener('click', function() {
        colorPaletteDropdown.classList.toggle('active');
      });
      document.querySelectorAll('#color-palette-dropdown .color-swatch').forEach(swatch => {
        swatch.addEventListener('click', function() {
          const color = this.getAttribute('data-color');
          fillColor = hexToRgb(color);
          colorPaletteDropdown.classList.remove('active');
        });
      });
      const shapesDropdown = document.getElementById('shapes-dropdown');
      document.getElementById('shapes').addEventListener('click', function() {
        shapesDropdown.classList.toggle('active');
      });
      document.querySelectorAll('#shapes-dropdown .dropdown-item').forEach(item => {
        item.addEventListener('click', function() {
          currentShape = this.getAttribute('data-shape');
          shapesDropdown.classList.remove('active');
        });
      });
      function hexToRgb(hex) {
        const r = parseInt(hex.slice(1, 3), 16);
        const g = parseInt(hex.slice(3, 5), 16);
        const b = parseInt(hex.slice(5, 7), 16);
        return [r, g, b, 255];
      }
      function getMousePos(canvas, evt) {
        const rect = canvas.getBoundingClientRect();
        return {
          x: evt.clientX - rect.left,
          y: evt.clientY - rect.top
        };
      }
    });
  </script>
</body>
</html>
