# Pixels and Resolution

This guide covers the fundamentals of digital display elements, how colors are produced, and how resolutions are compared and calculated.

---

## 1. What is a Pixel?
A **pixel** (short for *picture element*) is the smallest controllable physical point of light on a digital display. Millions of pixels are arranged in a grid to form the images, videos, and text you see on screens.

### RGB and Subpixels: How a Pixel Forms
Although a pixel looks like a single dot of a specific color, it is actually composed of three smaller components called **subpixels**:
*   🔴 **Red (R)**
*   🟢 **Green (G)**
*   🔵 **Blue (B)**

These three colors correspond to the human eye's cone receptors for color vision. By adjusting the intensity of light emitted by each of these three subpixels, a pixel can produce a vast spectrum of colors through **additive color mixing**:
*   **White:** All three subpixels (RGB) are turned on at maximum brightness.
*   **Black:** All three subpixels are turned off completely (no light).
*   **Yellow:** Red and Green subpixels are on, while Blue is off.
*   **Gray:** All three subpixels are on at an equal, medium intensity.

---

## 2. Calculating Pixels
The resolution of a screen is represented by the number of pixels along its width (horizontal) and height (vertical). 

$$\text{Total Pixels} = \text{Horizontal Pixels} \times \text{Vertical Pixels}$$

### Example Calculation
For a standard Full HD (1080p) screen:
*   **Width:** 1,920 pixels
*   **Height:** 1,080 pixels
*   **Calculation:** $1920 \times 1080 = 2,073,600\text{ pixels}$ (approximately **2.07 Megapixels**).

---

## 3. Resolution Comparison
Resolution dictates image clarity. Higher resolutions fit more pixels into the same screen area, creating a sharper, more detailed image (measured in **PPI** - Pixels Per Inch).

| Resolution Name | Common Terms / Marketing Names | Aspect Ratio | Dimensions (Width × Height) | Total Pixel Count | Megapixels |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **720p** | HD (High Definition) | 16:9 | $1280 \times 720$ | 921,600 | ~0.9 MP |
| **1080p** | FHD (Full HD) | 16:9 | $1920 \times 1080$ | 2,073,600 | ~2.1 MP |
| **2K** | DCI 2K (Cinema) / QHD (Quad HD)* | 17:9 / 16:9 | $2048 \times 1080$ / $2560 \times 1440$ | 2,211,840 / 3,686,400 | ~2.2 MP / ~3.7 MP |
| **4K** | UHD (Ultra HD) / DCI 4K | 16:9 / 17:9 | $3840 \times 2160$ / $4096 \times 2160$ | 8,294,400 / 8,847,360 | ~8.3 MP / ~8.8 MP |

> [!NOTE]
> **The 2K Clarification:** In the cinema industry, **2K** refers strictly to $2048 \times 1080$. However, in consumer electronics, computer monitors, and gaming, **1440p** ($2560 \times 1440$, also called **QHD** or Quad HD) is often casually referred to as "2K" because its width falls roughly halfway between 1080p and 4K.
