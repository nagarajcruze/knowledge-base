# Personal Knowledge Base 🧠

Welcome to my personal knowledge base! This repository acts as a single source of truth for everything I have learned, practiced, and experienced across software engineering, system administration, cloud technologies, personal finance, and hobbies.

---

## 📂 Repository Structure

The knowledge base is organized into several key areas:
- **DevOps**: Infrastructure, automation (Ansible, Terraform), containers (Docker, Podman), Kubernetes, CI/CD pipelines, SRE, security, and AWS.
- **Linux**: Operating system fundamentals, advanced administration, diagnostics, and shell scripting.
- **Network**: Core networking, subnetting, and DNS.
- **Homelab**: Self-hosting tutorials, hardware setup, storage systems (ZFS, GlusterFS), and virtualization.
- **LLMs**: Local Large Language Model installation, optimization, GGUF formats, and Ollama.
- **Entertainment & Hobbies**: Media displays, audiophile gear, motorbike maintenance, and motorsports.
- **Finance**: Personal wealth building, mutual funds, and stock markets.

---

## 🗃️ About `manifest.json`

The [`manifest.json`] file serves as the structured index/catalog of this repository. It defines categories, groups, and topics, allowing external scripts, static site builders, or documentation readers to automatically index and present the markdown files.

### Schema Structure
The manifest follows a tree hierarchy:
1. **Categories** (e.g., DevOps, Linux): Needs an `id`, `label`, `icon`, and `description`.
2. **Groups** (e.g., Containers, SRE): A subgroup within a category, having a `label` and `icon`.
3. **Topics** (e.g., Podman Guide, Linux Troubleshooting): The individual document entries containing:
   - `id`: A unique identifier string.
   - `label`: The human-readable title of the document.
   - `file`: The relative path to the `.md` file from the repository root.

---

## 📝 How to Add New Documents

To add a new document to the knowledge base:

1. **Create the Markdown File**:
   Write your content in a `.md` file inside the appropriate directory (e.g., `linux/my-new-guide.md`).

2. **Register in `manifest.json`**:
   Open [`manifest.json`] and locate the relevant category and group. Append your new document to the `topics` array:

   ```json
   {
     "id": "my-new-guide-id",
     "label": "My New Guide Title",
     "file": "linux/my-new-guide.md"
   }
   ```

3. **Verify the Paths**:
   Ensure the `"file"` path exactly matches the relative path to your markdown file using forward slashes (`/`), even on Windows.
