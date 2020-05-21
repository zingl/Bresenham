# The Beauty of Bresenham's Algorithm
This repository introduces a compact and efficient implementation of Bresenham's algorithm to plot lines, circles, ellipses and Bézier curves.
## The Algorithm
A simple implementation to plot lines, circles, ellipses and Bézier curves.

A detailed documentation of the algorithm and more program examples are availble at [GitHub](https://zingl.github.io/bresenham.html).
## Line
A simple example of Bresenham's line algorithm.
```c
void plotLine(int x0, int y0, int x1, int y1)
{
   int dx =  abs(x1-x0), sx = x0<x1 ? 1 : -1;
   int dy = -abs(y1-y0), sy = y0<y1 ? 1 : -1; 
   int err = dx+dy, e2; /* error value e_xy */
 
   for(;;){  /* loop */
      setPixel(x0,y0);
      if (x0==x1 && y0==y1) break;
      e2 = 2*err;
      if (e2 >= dy) { err += dy; x0 += sx; } /* e_xy+e_x > 0 */
      if (e2 <= dx) { err += dx; y0 += sy; } /* e_xy+e_y < 0 */
   }
}
```
## Bresenham in 3D
The algorithm could be extended to three (or more) dimensions.
```c
void plotLine3d(int x0, int y0, int z0, int x1, int y1, int z1)
{
   int dx = abs(x1-x0), sx = x0<x1 ? 1 : -1;
   int dy = abs(y1-y0), sy = y0<y1 ? 1 : -1; 
   int dz = abs(z1-z0), sz = z0<z1 ? 1 : -1; 
   int dm = max(dx,dy,dz), i = dm; /* maximum difference */
 
   for(x1 = y1 = z1 = i/2; i-- >= 0; ) {  /* loop */
      setPixel(x0,y0,z0);
      x1 -= dx; if (x1 < 0) { x1 += dm; x0 += sx; } 
      y1 -= dy; if (y1 < 0) { y1 += dm; y0 += sy; } 
      z1 -= dz; if (z1 < 0) { z1 += dm; z0 += sz; } 
   }
}
```
## Circle
This is an implementation of the circle algorithm.
```c
void plotCircle(int xm, int ym, int r)
{
   int x = -r, y = 0, err = 2-2*r; /* II. Quadrant */ 
   do {
      setPixel(xm-x, ym+y); /*   I. Quadrant */
      setPixel(xm-y, ym-x); /*  II. Quadrant */
      setPixel(xm+x, ym-y); /* III. Quadrant */
      setPixel(xm+y, ym+x); /*  IV. Quadrant */
      r = err;
      if (r <= y) err += ++y*2+1;           /* e_xy+e_y < 0 */
      if (r > x || err > y) err += ++x*2+1; /* e_xy+e_x > 0 or no 2nd y-step */
   } while (x < 0);
}
```
## Ellipse
This program example plots an ellipse inside a specified rectangle.
```c
void plotEllipseRect(int x0, int y0, int x1, int y1)
{
   int a = abs(x1-x0), b = abs(y1-y0), b1 = b&1; /* values of diameter */
   long dx = 4*(1-a)*b*b, dy = 4*(b1+1)*a*a; /* error increment */
   long err = dx+dy+b1*a*a, e2; /* error of 1.step */

   if (x0 > x1) { x0 = x1; x1 += a; } /* if called with swapped points */
   if (y0 > y1) y0 = y1; /* .. exchange them */
   y0 += (b+1)/2; y1 = y0-b1;   /* starting pixel */
   a *= 8*a; b1 = 8*b*b;

   do {
       setPixel(x1, y0); /*   I. Quadrant */
       setPixel(x0, y0); /*  II. Quadrant */
       setPixel(x0, y1); /* III. Quadrant */
       setPixel(x1, y1); /*  IV. Quadrant */
       e2 = 2*err;
       if (e2 <= dy) { y0++; y1--; err += dy += a; }  /* y step */ 
       if (e2 >= dx || 2*err > dy) { x0++; x1--; err += dx += b1; } /* x step */
   } while (x0 <= x1);
   
   while (y0-y1 < b) {  /* too early stop of flat ellipses a=1 */
       setPixel(x0-1, y0); /* -> finish tip of ellipse */
       setPixel(x1+1, y0++); 
       setPixel(x0-1, y1);
       setPixel(x1+1, y1--); 
   }
}
```
## Bézier curve
This program example plots a quadratic Bézier curve limited to gradients without sign change.
```c
void plotQuadBezierSeg(int x0, int y0, int x1, int y1, int x2, int y2)
{                            
  int sx = x2-x1, sy = y2-y1;
  long xx = x0-x1, yy = y0-y1, xy;         /* relative values for checks */
  double dx, dy, err, cur = xx*sy-yy*sx;                    /* curvature */

  assert(xx*sx <= 0 && yy*sy <= 0);  /* sign of gradient must not change */

  if (sx*(long)sx+sy*(long)sy > xx*xx+yy*yy) { /* begin with longer part */ 
    x2 = x0; x0 = sx+x1; y2 = y0; y0 = sy+y1; cur = -cur;  /* swap P0 P2 */
  }  
  if (cur != 0) {                                    /* no straight line */
    xx += sx; xx *= sx = x0 < x2 ? 1 : -1;           /* x step direction */
    yy += sy; yy *= sy = y0 < y2 ? 1 : -1;           /* y step direction */
    xy = 2*xx*yy; xx *= xx; yy *= yy;          /* differences 2nd degree */
    if (cur*sx*sy < 0) {                           /* negated curvature? */
      xx = -xx; yy = -yy; xy = -xy; cur = -cur;
    }
    dx = 4.0*sy*cur*(x1-x0)+xx-xy;             /* differences 1st degree */
    dy = 4.0*sx*cur*(y0-y1)+yy-xy;
    xx += xx; yy += yy; err = dx+dy+xy;                /* error 1st step */    
    do {                              
      setPixel(x0,y0);                                     /* plot curve */
      if (x0 == x2 && y0 == y2) return;  /* last pixel -> curve finished */
      y1 = 2*err < dx;                  /* save value for test of y step */
      if (2*err > dy) { x0 += sx; dx -= xy; err += dy += yy; } /* x step */
      if (    y1    ) { y0 += sy; dy -= xy; err += dx += xx; } /* y step */
    } while (dy < dx );           /* gradient negates -> algorithm fails */
  }
  plotLine(x0,y0, x2,y2);                  /* plot remaining part to end */
}  
```
## Features of the rasterising algorithm:
* __Universal__: 
 This algorithm plots lines, circles, ellipses, Bézier curves and more
* __Fast__: 
 Draws complex curves nearly as fast as lines.
* __Simple__: 
 Short and compact implementation.
* __Exact__:
 No approximation of the curve.
* __Smooth__:
 Apply anti-aliasing to any curve.
* __Flexible__:
 Adjustable line thickness.
* __Open source__:
 Free software program (MIT license).

The principle of the algorithm could be used to rasterize any polynomial curve.
