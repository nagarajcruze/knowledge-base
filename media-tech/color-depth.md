# Color Depth: 8-Bit vs. 10-Bit

Color depth (also known as **bit depth**) determines how many unique colors a display can show or a camera can record. It refers to the number of bits used to represent the color of a single pixel.

---

## 1. What is Color Depth?
Digital images use the **RGB color model**. To display a color, a pixel mixes Red, Green, and Blue light. The color depth tells us how many discrete shades (gradations) of *each* color channel are available. 

---

## 2. 8-Bit Color Depth
In an 8-bit system, there are 8 bits of data allocated for each of the three color channels (Red, Green, Blue).

*   **Shades per channel:** $2^8 = 256$ shades of Red, 256 of Green, and 256 of Blue.
*   **Total Color Palette:** $256 \times 256 \times 256 = 16,777,216$ colors (**~16.7 Million**).

### The Limitation: Color Banding
Because there are only 256 steps between the darkest and brightest shades, you will sometimes see distinct steps or "bands" of colors when displaying smooth gradients (like a sunset, clear blue sky, or shadows in a dark scene). This visual artifact is called **color banding**.

---

## 3. 10-Bit Color Depth (Deep Color)
A 10-bit system allocates 10 bits of data per color channel. 

*   **Shades per channel:** $2^{10} = 1,024$ shades of Red, 1,024 of Green, and 1,024 of Blue (4x more shades per channel than 8-bit).
*   **Total Color Palette:** $1024 \times 1024 \times 1024 = 1,073,741,824$ colors (**~1.07 Billion**).

### The Solution: Smooth Gradients
By jumping from 16.7 million colors to over 1 billion, the transition between color shades becomes so fine that the human eye cannot distinguish the individual steps. This completely eliminates color banding, resulting in hyper-realistic images.

---

## 4. Summary Comparison

| Feature | 8-Bit Color | 10-Bit Color |
| :--- | :--- | :--- |
| **Shades Per Channel** | 256 | 1,024 (4x increase) |
| **Total Colors** | ~16.7 Million | ~1.07 Billion (64x increase) |
| **Gradients & Shadows** | Prone to color banding | Seamlessly smooth transitions |
| **HDR Suitability** | Poor (Not recommended for HDR) | **Required** (Core component of HDR10/Dolby Vision) |
| **File / Stream Size** | Standard sizes | Roughly 20% to 25% larger file sizes before compression |
| **Use Cases** | SDR TV, standard web browsing, JPG photos | UHD Blu-rays, professional video editing, RAW photography, HDR gaming |

> [!TIP]
> **8-Bit + FRC (Temporal Dithering):** Some budget monitors are marketed as "10-bit" but are actually **8-bit + FRC (Frame Rate Control)**. FRC flashes two different color shades rapidly in consecutive frames to trick the human eye into perceiving a shade that the panel cannot physically display. While not as perfect as a native 10-bit panel, it is significantly cheaper and offers a good approximation of 10-bit color.
