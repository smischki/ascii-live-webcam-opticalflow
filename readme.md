# Optical Flow Arrow Mirror
 
A live webcam mirror that detects motion and draws it as ASCII arrows pointing the way things move, in real time, right in the browser.
 
It uses the Horn–Schunck optical flow algorithm to work out how every pixel moves between two webcam frames. Still areas stay empty, and moving areas fill with arrows (`→ ↘ ↓ ↙ ← ↖ ↑ ↗`) that follow your movement. 

## How it works
 
Every frame goes through four passes. Each pass has to finish before the next one starts, because each one needs results from neighbouring pixels.
 
1. **Brightness:** each frame is shrunk onto a small hidden `<canvas>`, so one pixel equals one character. Every pixel's red, green and blue values are combined into a single brightness from 0 to 1. The first frame is only stored, because there is nothing to compare it with yet.
2. **Brightness changes:** for every pixel, three values are computed:
   - `Ix` and `Iy`: how the brightness changes from left to right and from top to bottom, using Sobel kernels (divided by 8 to give the change per pixel)
   - `It`: how the brightness changed over time, which is this frame minus the previous frame
```
sobel_x          sobel_y
-1  0  1         -1 -2 -1
-2  0  2          0  0  0
-1  0  1          1  2  1
```
 
3. **Horn–Schunck:** the goal is to find the motion `u` (horizontal) and `v` (vertical) of every pixel, in pixels per frame. The update below is repeated `ITERATIONS` times for every pixel:
```
u_avg, v_avg = weighted average of the neighbours' motion
common = (Ix·u_avg + Iy·v_avg + It) / (alpha² + Ix² + Iy²)
u = u_avg − Ix · common
v = v_avg − Iy · common
```
 
   The neighbour average uses the following kernel:
 
```
1/12  1/6  1/12
1/6    0   1/6
1/12  1/6  1/12
```
 
4. **Draw arrows:** for each pixel, the speed is `√(u² + v²)`. Pixels slower than `MIN_SPEED` become a space. For the others, `Math.atan2(v, u)` gives the direction of motion as an angle from 0° to 360°. The full circle is split into 8 slices of 45°, and each slice gets its arrow.


## Settings
 
Change these at the top of the `<script>`: COLS, alpha, ITERATIONS, MIN_SPEED, MAX_SPEED, ARROWS.
 


 
