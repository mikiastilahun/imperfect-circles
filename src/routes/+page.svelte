<script lang="ts">
  import { onMount } from "svelte";

  let canvas: HTMLCanvasElement | null = null;

  let context: CanvasRenderingContext2D | null;
  let displayWidth: number;
  let displayHeight: number;

  onMount(() => {
    setTimeout(() => {
      generate();
    }, 1000);
  });

  $: if (canvas) {
    context = canvas?.getContext("2d");
    displayWidth = canvas?.width;
    displayHeight = canvas?.height;
  }

  function generate() {
    if (context == null) return;
    context.clearRect(0, 0, displayWidth, displayHeight);
    let centerX, centerY;

    let color;

    let minRad, maxRad;
    let phase;

    let radials = 100; // set the number of radials you want.

    let circleArray: any = [];
    for (let i = 0; i < radials; i++) {
      centerX = Math.random() * (displayWidth - 2);
      centerY = Math.random() * (displayHeight - 2);
      maxRad = Math.random() * 0.1 * displayWidth;
      minRad = 0.88 * maxRad;
      phase = Math.random() * Math.PI * 2;
      color = "rgba(0r, 0g, 0b, 0.5) ";

      // if the new circle is not overlapping any existing circles, add it to the array
      let newCircle = { x: centerX, y: centerY, maxRad: maxRad };

      // assume rectangle overlap
      let overlap = circleArray.some(
        (existing: any) =>
          Math.abs(existing.x - newCircle.x) <
            existing.maxRad + newCircle.maxRad &&
          Math.abs(existing.y - newCircle.y) <
            existing.maxRad + newCircle.maxRad
      );

      if (overlap) {
        console.log("overlap");
        continue;
      }
      drawCircle(centerX, centerY, minRad, maxRad, phase, color);
      circleArray.push(newCircle);
    }
  }

  function drawCircle(
    centerX: number,
    centerY: number,
    minRad: number,
    maxRad: number,
    phase: number,
    color: string
  ) {
    setInterval(() => {
      if (context == null) return;
      context.clearRect(
        centerX - maxRad,
        centerY - maxRad,
        2 * maxRad,
        2 * maxRad
      );

      let point;
      let rad, theta;
      let twoPi = 2 * Math.PI;
      let x0, y0;

      //generate the random function that will be used to lety the radius, 9 iterations of subdivision
      let pointList = setLinePoints(9);

      context.strokeStyle = color;
      context.lineWidth = 1.01;
      context.fillStyle = color;
      context?.beginPath();
      point = pointList.first;
      theta = phase;
      rad = minRad + point.y * (maxRad - minRad);
      x0 = centerX + rad * Math.cos(theta);
      y0 = centerY + rad * Math.sin(theta);
      context?.lineTo(x0, y0);
      while (point.next != null) {
        point = point.next;
        theta = twoPi * point.x + phase;
        rad = minRad + point.y * (maxRad - minRad);
        x0 = centerX + rad * Math.cos(theta);
        y0 = centerY + rad * Math.sin(theta);
        context?.lineTo(x0, y0);
      }
      context?.stroke();
      context?.fill();
    }, 100);
  }

  function setLinePoints(iterations: number) {
    let pointList: any = {};
    pointList.first = { x: 0, y: 1 };
    let lastPoint = { x: 1, y: 1 };
    let minY = 1;
    let maxY = 1;
    let point;
    let nextPoint;
    let dx, newX, newY;

    pointList.first.next = lastPoint;
    for (let i = 0; i < iterations; i++) {
      point = pointList.first;
      while (point.next != null) {
        nextPoint = point.next;

        dx = nextPoint.x - point.x;
        newX = 0.5 * (point.x + nextPoint.x);
        newY = 0.5 * (point.y + nextPoint.y);
        newY += dx * (Math.random() * 2 - 1);

        let newPoint: any = { x: newX, y: newY };

        //min, max
        if (newY < minY) {
          minY = newY;
        } else if (newY > maxY) {
          maxY = newY;
        }

        //put between points
        newPoint.next = nextPoint;
        point.next = newPoint;

        point = nextPoint;
      }
    }

    //normalize to values between 0 and 1
    if (maxY != minY) {
      let normalizeRate = 1 / (maxY - minY);
      point = pointList.first;
      while (point != null) {
        point.y = normalizeRate * (point.y - minY);
        point = point.next;
      }
    }
    //unlikely that max = min, but could happen if using zero iterations. In this case, set all points equal to 1.
    else {
      point = pointList.first;
      while (point != null) {
        point.y = 1;
        point = point.next;
      }
    }

    return pointList;
  }
</script>

<div class="w-screen h-screen flex justify-center items-center">
  <div>
    <canvas bind:this={canvas} width="1000px" height="1000px">
      This browser does not support the canvas element.
    </canvas>
  </div>
</div>

<style>
</style>
