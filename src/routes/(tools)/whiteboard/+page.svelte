<script>
  // @ts-nocheck
  import { onMount } from "svelte";

  let image = null;
  let isDrawing = false;
  let lastX = 0;
  let lastY = 0;
  let tool = "pencil";
  let eraserSize = 10;
  let drawingHistory = [];
  let historyIndex = -1;
  let fillColor = [0, 0, 0, 255];
  let shape = null;
  let startX, startY;

  onMount(() => {
    function resizeCanvas() {
      let canvas = document.getElementById("canvas");
      const ctx = canvas.getContext("2d");
      canvas.width = canvas.offsetWidth;
      canvas.height = canvas.offsetHeight;
      if (drawingHistory.length > 0) {
        ctx.putImageData(drawingHistory[historyIndex], 0, 0);
      }
    }

    resizeCanvas();
    window.addEventListener("resize", resizeCanvas);
  });

  function startDrawing(e) {
    isDrawing = true;
    [lastX, lastY] = [e.offsetX, e.offsetY];
    if (tool === "shapes" && shape) {
      startX = e.offsetX;
      startY = e.offsetY;
    }
  }

  function draw(e) {
    let canvas = document.getElementById("canvas");
    const ctx = canvas.getContext("2d");
    if (!isDrawing) return;
    ctx.lineWidth = tool === "eraser" ? eraserSize : 2;
    ctx.lineCap = "round";
    ctx.strokeStyle = tool === "eraser" ? "#fff" : "#000";

    if (tool === "shapes" && shape) {
      // Redraw the last state of the canvas to avoid drawing artifacts
      if (drawingHistory.length > 0) {
        ctx.putImageData(drawingHistory[historyIndex], 0, 0);
      }
      switch (shape) {
        case "circle":
          drawCircle(startX, startY, e.offsetX, e.offsetY);
          break;
        case "square":
          drawSquare(startX, startY, e.offsetX, e.offsetY);
          break;
        case "line":
          drawLine(startX, startY, e.offsetX, e.offsetY);
          break;
      }
    } else {
      ctx.beginPath();
      ctx.moveTo(lastX, lastY);
      ctx.lineTo(e.offsetX, e.offsetY);
      ctx.stroke();
      [lastX, lastY] = [e.offsetX, e.offsetY];
    }
  }

  function stopDrawing() {
    if (!isDrawing) return;
    isDrawing = false;
    if (tool !== "eraser" && tool !== "shapes") {
      saveDrawingAction();
    } else if (tool === "shapes" && shape) {
      saveDrawingAction();
    }
  }

  function saveDrawingAction() {
    let canvas = document.getElementById("canvas");
    const ctx = canvas.getContext("2d");
    if (historyIndex < drawingHistory.length - 1) {
      drawingHistory = drawingHistory.slice(0, historyIndex + 1);
    }
    drawingHistory.push(ctx.getImageData(0, 0, canvas.width, canvas.height));
    historyIndex++;
  }

  function undoDrawing() {
    let canvas = document.getElementById("canvas");
    const ctx = canvas.getContext("2d");
    if (historyIndex > 0) {
      historyIndex--;
      ctx.putImageData(drawingHistory[historyIndex], 0, 0);
    }
  }

  function redoDrawing() {
    let canvas = document.getElementById("canvas");
    const ctx = canvas.getContext("2d");
    if (historyIndex < drawingHistory.length - 1) {
      historyIndex++;
      ctx.putImageData(drawingHistory[historyIndex], 0, 0);
    }
  }

  function floodFill(x, y, fillColor) {
    let canvas = document.getElementById("canvas");
    const ctx = canvas.getContext("2d");
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data = imageData.data;
    const targetColor = getColorAtPixel(data, x, y);

    if (colorsMatch(targetColor, fillColor)) return;

    const stack = [[x, y]];

    while (stack.length) {
      const [currentX, currentY] = stack.pop();
      const currentPos = (currentY * canvas.width + currentX) * 4;

      if (!colorsMatch(getColorAtPixel(data, currentX, currentY), targetColor))
        continue;

      setColorAtPixel(data, currentX, currentY, fillColor);

      stack.push([currentX + 1, currentY]);
      stack.push([currentX - 1, currentY]);
      stack.push([currentX, currentY + 1]);
      stack.push([currentX, currentY - 1]);
    }

    ctx.putImageData(imageData, 0, 0);
    saveDrawingAction();
  }

  function getColorAtPixel(data, x, y) {
    let canvas = document.getElementById("canvas");
    const pos = (y * canvas.width + x) * 4;
    return [data[pos], data[pos + 1], data[pos + 2], data[pos + 3]];
  }

  function setColorAtPixel(data, x, y, color) {
    let canvas = document.getElementById("canvas");
    let pos = (y * canvas.width + x) * 4;
    [data[pos], data[pos + 1], data[pos + 2], data[pos + 3]] = color;
  }

  function colorsMatch(color1, color2) {
    return (
      color1[0] === color2[0] &&
      color1[1] === color2[1] &&
      color1[2] === color2[2] &&
      color1[3] === color2[3]
    );
  }

  function activateTool(newTool) {
    let canvas = document.getElementById("canvas");
    tool = newTool;
    canvas.style.cursor =
      tool === "eraser"
        ? "url(\"data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 20 20' fill='%23ffffff'%3E%3Cpath fill-rule='evenodd' d='M5 13V6h10v7H5zm11 1v5a1 1 0 0 1-1 1H5a1 1 0 0 1-1-1v-5h12zM7 8v2h6V8H7z' clip-rule='evenodd'/%3E%3C/svg%3E\") 14 14, auto"
        : "crosshair";
  }

  function toggleDropdown(dropdown) {
    const isActive = dropdown.classList.toggle("active");
    dropdown.parentElement
      .querySelector(".dropdown-arrow")
      .classList.toggle("active", isActive);
  }

  function drawCircle(x1, y1, x2, y2) {
    let canvas = document.getElementById("canvas");
    const ctx = canvas.getContext("2d");
    const radius = Math.sqrt((x2 - x1) ** 2 + (y2 - y1) ** 2);
    ctx.beginPath();
    ctx.arc(x1, y1, radius, 0, Math.PI * 2);
    ctx.stroke();
  }

  function drawSquare(x1, y1, x2, y2) {
    let canvas = document.getElementById("canvas");
    const ctx = canvas.getContext("2d");
    const side = Math.max(Math.abs(x2 - x1), Math.abs(y2 - y1));
    ctx.strokeRect(x1, y1, side, side);
  }

  function drawLine(x1, y1, x2, y2) {
    let canvas = document.getElementById("canvas");
    const ctx = canvas.getContext("2d");
    ctx.beginPath();
    ctx.moveTo(x1, y1);
    ctx.lineTo(x2, y2);
    ctx.stroke();
  }

  function hexToRgb(hex) {
    const bigint = parseInt(hex.slice(1), 16);
    const r = (bigint >> 16) & 255;
    const g = (bigint >> 8) & 255;
    const b = bigint & 255;
    return [r, g, b];
  }

  function showImg() {
    const userInput = document.getElementById("imageInp");
    console.log(userInput);
    let file = userInput.files[0];
    console.log(file);
    let dpi = window.devicePixelRatio;
    let canvas = document.getElementById("canvas");
    const ctx = canvas.getContext("2d");

    function fix_dpi() {
      let style_height = +getComputedStyle(canvas)
        .getPropertyValue("height")
        .slice(0, -2);
      let style_width = +getComputedStyle(canvas)
        .getPropertyValue("width")
        .slice(0, -2);
      canvas.setAttribute("height", style_height * dpi);
      canvas.setAttribute("width", style_width * dpi);
    }

    if (file) {
      let imgURL = URL.createObjectURL(file);
      const img = new Image();
      img.onload = function () {
        fix_dpi();
        var hRatio = canvas.width / img.width;
        var vRatio = canvas.height / img.height;
        var ratio = Math.min(hRatio, vRatio);
        var centerShift_x = (canvas.width - img.width * ratio) / 2;
        var centerShift_y = (canvas.height - img.height * ratio) / 2;
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.drawImage(
          img,
          0,
          0,
          img.width,
          img.height,
          centerShift_x,
          centerShift_y,
          img.width * ratio,
          img.height * ratio
        );
      };
      img.src = imgURL;
      resizeCanvas();
    } else {
      alert("No file detected, please insert a file.");
    }
  }

  function clearCanvas() {
    const canvas = document.getElementById("canvas");
    const ctx = canvas.getContext("2d");
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    image = null;
    document.getElementById("imageInp").value = "";
  }

  function downloadCanvas() {
    const canvas = document.getElementById("canvas");
    const dataUrl = canvas.toDataURL("image/jpeg");
    const a = document.createElement("a");
    document.body.appendChild(a);
    a.href = dataUrl;
    a.download = "Canvas.jpg";
    a.click();
    document.body.removeChild(a);
  }
</script>

<main class="w-full h-[600px]">
  <div class="h-full w-full z-20 flex flex-col rounded-xl">
    <div
      class="h-[9%] flex flex-row gap-2 bg-gray-300 p-3 rounded-tr-xl rounded-tl-xl"
    >
      <button
        class="border border-black p-1 rounded-lg h-full w-auto hover:bg-gray-400"
        on:click={activateTool("pencil")}
        id="pencil"><img src="#" alt="pencil" /></button
      >
      <button
        class="group border border-black p-1 rounded-lg h-full w-auto hover:bg-gray-400 relative"
        id="eraser"
        on:click={activateTool(tool === "eraser" ? "pencil" : "eraser")}
      >
        <img src="#" alt="eraser" />
        <input
          class="hidden absolute translate-y-[40px] z-10 focus:block group-visited:block group-"
          id="sliderSelection"
          type="range"
          step="2"
          value="5"
          min="2"
          max="20"
        />
      </button>
      <button
        class="border border-black p-1 rounded-lg h-full w-auto hover:bg-gray-400"
        on:click={activateTool("fill")}
        id="fill"><img src="#" alt="paint" /></button
      >
      <button
        class="border border-black p-1 rounded-lg h-full w-auto hover:bg-gray-400"
        on:click={activateTool("shapes")}
        id="shapes"><img src="#" alt="shape" /></button
      >
      <div
        class="border border-black p-1 rounded-lg h-full w-20 hover:bg-gray-400 relative flex flex-col item-center justify-center"
      >
        <img src="#" alt="img" class="absolute" />
        <input
          name="imageInp"
          id="imageInp"
          type="file"
          accept="image/png, image/jpeg"
          class="w-full h-full opacity-0 z-10 cursor-pointer"
          bind:files={image}
          on:change={() => {
            console.log("Change detected");
            showImg();
          }}
          required
        />
      </div>
      <button
        class="border border-black p-1 rounded-lg h-full w-auto hover:bg-gray-400"
        on:click={undoDrawing()}
        id="undo"><img src="#" alt="undo" /></button
      >
      <button
        class="border border-black p-1 rounded-lg h-full w-auto hover:bg-gray-400"
        on:click={redoDrawing()}
        id="redo"><img src="#" alt="redo" /></button
      >
      <button
        class="border border-black p-1 rounded-lg h-full w-auto hover:bg-gray-400"
        id="clear"
        on:click={clearCanvas}><img src="#" alt="clear" /></button
      >
      <!-- svelte-ignore a11y-invalid-attribute -->
      <button
        id="canvasDownload"
        on:click={downloadCanvas}
        class="border border-black p-1 rounded-lg h-full w-auto hover:bg-gray-400"
        ><img src="#" alt="download" /></button
      >
    </div>
    <!-- svelte-ignore a11y-mouse-events-have-key-events -->
    <canvas
      id="canvas"
      class="border border-black h-full w-full bg-white"
      on:mousedown={startDrawing}
      on:mousemove={draw}
      on:mouseup={stopDrawing}
      on:mouseout={stopDrawing}
    ></canvas>
  </div>
</main>

<style>
</style>
