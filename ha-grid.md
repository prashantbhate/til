# ha-grid Highly Available Grid !!

I needed to match and the fixma dimentions in the html , there was always option to install onscreen ruler to measure the component dimentions
with figma design but where is fun in that.

So here is what I did, Idea was to crate create a 50x50 pixel overlay grid with 10px gridlines that can be draggable on the screen to test

below is the function, copy pasted in chrome dev tools console , once function is created it can be called to get a dragable grid

```
// Default 50x50 red box with 10px grid
createGridBox();

// A green 80x80 box with 20px grid
createGridBox({ boxSize: 80, gridSize: 20, lineColor: "rgba(0,128,0,0.8)", fillColor: "rgba(0,255,0,0.15)" });

// A blue tiny checkerboard (5px grid)
createGridBox({ boxSize: 50, gridSize: 5, lineColor: "rgba(0,0,255,0.7)", fillColor: "rgba(0,0,255,0.1)" });

```


```
function createGridBox({
  boxSize = 50,           // total square size in px
  gridSize = 10,          // spacing between grid lines
  lineColor = "rgba(0,0,0,0.5)", // color of grid lines
  fillColor = "rgba(255,0,0,0.1)" // background fill color
} = {}) {
  const ID = "pixel-grid-" + Math.random().toString(36).slice(2, 7);

  // create square
  const el = document.createElement("div");
  el.id = ID;
  Object.assign(el.style, {
    width: boxSize + "px",
    height: boxSize + "px",
    position: "fixed",
    top: "10px",
    left: "10px",
    zIndex: "2147483647",
    cursor: "move",
    backgroundImage:
      `linear-gradient(${lineColor} 1px, transparent 1px),
       linear-gradient(90deg, ${lineColor} 1px, transparent 1px)`,
    backgroundSize: `${gridSize}px ${gridSize}px`,
    backgroundColor: fillColor,
    boxSizing: "border-box",
    border: "1px solid rgba(0,0,0,0.2)",
    touchAction: "none"
  });
  document.body.appendChild(el);

  // drag logic
  let dragging = false, offX = 0, offY = 0;
  const startDrag = (clientX, clientY) => {
    dragging = true;
    offX = clientX - el.offsetLeft;
    offY = clientY - el.offsetTop;
  };
  const moveTo = (x, y) => {
    el.style.left = x + "px";
    el.style.top = y + "px";
  };

  el.addEventListener("mousedown", e => { startDrag(e.clientX, e.clientY); e.preventDefault(); });
  document.addEventListener("mousemove", e => { if (dragging) moveTo(e.clientX - offX, e.clientY - offY); });
  document.addEventListener("mouseup", () => { dragging = false; });

  // touch
  el.addEventListener("touchstart", e => { const t = e.touches[0]; startDrag(t.clientX, t.clientY); e.preventDefault(); }, {passive:false});
  document.addEventListener("touchmove", e => { if (dragging) { const t = e.touches[0]; moveTo(t.clientX - offX, t.clientY - offY); e.preventDefault(); } }, {passive:false});
  document.addEventListener("touchend", () => { dragging = false; });

  // keyboard nudge + remove
  const keyHandler = (e) => {
    if (!document.getElementById(ID)) {
      window.removeEventListener("keydown", keyHandler);
      return;
    }
    const step = e.shiftKey ? 10 : 1;
    switch (e.key) {
      case "ArrowLeft": moveTo(el.offsetLeft - step, el.offsetTop); e.preventDefault(); break;
      case "ArrowRight": moveTo(el.offsetLeft + step, el.offsetTop); e.preventDefault(); break;
      case "ArrowUp": moveTo(el.offsetLeft, el.offsetTop - step); e.preventDefault(); break;
      case "ArrowDown": moveTo(el.offsetLeft, el.offsetTop + step); e.preventDefault(); break;
      case "Escape": el.remove(); e.preventDefault(); break;
    }
  };
  window.addEventListener("keydown", keyHandler);

  return el;
}

```
