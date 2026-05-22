# HDR and Dolby Vision Formats

High Dynamic Range (HDR) is a technology that significantly improves image quality by expanding the contrast ratio and color palette of displays.

---

## 1. What is HDR?
Unlike Standard Dynamic Range (SDR), which has been the broadcast standard for decades, HDR allows a display to show:
1.  **Higher Contrast:** Brighter highlights (like sunlight or sparks) alongside deeper, richer shadows without losing detail.
2.  **Wider Color Gamut (WCG):** Displays can reproduce a wider spectrum of colors (such as deep reds and emerald greens) using spaces like **DCI-P3** or **Rec. 2020**.

---

## 2. Comparing HDR Standards
Different formats handle brightness levels, color depth, and **metadata** (the instructions embedded in video files that tell the display how to render brightness levels) differently.

| Standard | Metadata Type | Color Depth | Royalty / Licensing | Peak Brightness (Target) | Common Content Sources |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **HDR10** | Static | 10-bit | Free (Open Standard) | 1,000 nits | UHD Blu-ray, Netflix, YouTube, Gaming |
| **HDR10+** | Dynamic | 10-bit | Free (Open Standard) | Up to 4,000 nits | Prime Video, select UHD Blu-rays |
| **Dolby Vision** | Dynamic | 10-bit or 12-bit | Proprietary (Paid License) | Up to 10,000 nits | Netflix, Disney+, Apple TV+, UHD Blu-ray |
| **HLG** | None | 10-bit | Free (Open Standard) | SDR / HDR compatible | Live TV Broadcasts, BBC iPlayer, YouTube |

---

## 3. Deep Dive into Formats

### HDR10 (Baseline Standard)
*   **Static Metadata:** The video file establishes a single brightness limit for the entire movie. If a movie has a very bright scene (daytime) and a very dark scene (nighttime), the TV must use a single compromise setting, which can lead to crushed shadows or blown-out highlights.
*   **Compatibility:** The mandatory baseline format for all HDR displays.

### Dolby Vision (Premium Standard)
*   **Dynamic Metadata:** Brightness settings are updated **frame-by-frame** or **scene-by-scene**. The movie director can specify exactly how bright or dark each shot should look.
*   **12-bit Support:** While current consumer TVs are physically limited to 10-bit panels, Dolby Vision is future-proofed to support up to 12-bit color depth.
*   **Mastering:** Can be mastered up to 10,000 nits (typically mastered at 4,000 nits today).

### HDR10+ (The Open Competitor)
*   **Dynamic Metadata:** Developed by Samsung, Panasonic, and 20th Century Fox to compete with Dolby Vision.
*   **Royalty-free:** Offers frame-by-frame adjustments similar to Dolby Vision, but manufacturers do not have to pay licensing fees to Dolby.

### HLG (Hybrid Log-Gamma)
*   **Broadcast-Oriented:** Co-developed by the BBC (UK) and NHK (Japan).
*   **No Metadata:** Instead of relying on metadata, it encodes the standard light response curve (gamma) and a logarithmic curve (for high-brightness levels) into a single signal.
*   **Backwards Compatible:** Older SDR TVs can display an HLG broadcast feed normally (in SDR), while newer HDR TVs decode the extra high-brightness information. This saves precious bandwidth for television broadcasters.
