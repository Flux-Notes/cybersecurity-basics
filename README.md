# 🔐 Cybersecurity Basics Notes

> *"Security is not a product, but a process."*
> — **Bruce Schneier**

A structured, beginner-to-intermediate documentation for **Cybersecurity** — built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) and hosted on GitHub Pages.

![MkDocs](https://img.shields.io/badge/MkDocs-Material-blue?logo=materialformkdocs)
![GitHub Pages](https://img.shields.io/badge/Hosted-GitHub%20Pages-222?logo=github)
![License](https://img.shields.io/badge/License-MIT-green)
![Contributions Welcome](https://img.shields.io/badge/Contributions-Welcome-orange)

---

## 📚 Topics Covered

<details>
<summary>🛡️ Introduction to Cybersecurity</summary>

- Careers in Cyber
- Defensive Security Intro
- Offensive Security Intro

</details>

<details>
<summary>🌐 Network Fundamentals</summary>

- What is Networking
- Intro to LAN
- OSI Model
- Packets and Frames
- Extending Your Network

</details>

<details>
<summary>💻 Operating Systems</summary>

- Linux CLI Basics
- Operating System Security
- Windows Basics
- Windows CLI Basics

</details>

<details>
<summary>🌍 Web Fundamentals</summary>

- How Websites Work
- DNS
- HTTP & HTTPS
- Web Request Flow

</details>

<details>
<summary>⚔️ Attacks & Defenses</summary>

- CIA Triad
- Cryptography
- Become a Hacker
- Become a Defender

</details>

---

## 🛠️ Built With

| Tool | Purpose |
|------|---------|
| [MkDocs](https://www.mkdocs.org/) | Static site generator |
| [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) | Theme |
| [GitHub Pages](https://pages.github.com/) | Hosting |
| Markdown | Content writing |

---

## 🚀 Run Locally

Follow these steps to run the project on your machine:

### 1. Clone the repo
```bash
git clone https://github.com/your-username/Cybersecurity-Basics.git
cd Cybersecurity-Basics
```

### 2. Create & activate virtual environment
```bash
python -m venv venv

# Mac/Linux
source venv/bin/activate

# Windows
venv\Scripts\activate
```

### 3. Install dependencies
```bash
pip install mkdocs mkdocs-material
```

### 4. Preview locally
```bash
mkdocs serve
```
Open [http://127.0.0.1:8000](http://127.0.0.1:8000) in your browser.

### 5. Deploy to GitHub Pages
```bash
mkdocs gh-deploy
```

---

## 📁 Project Structure

```
Cybersecurity-Basics/
├── docs/
│   ├── index.md
│   ├── introduction-to-cybersecurity/
│   │   ├── careers-in-cyber.md
│   │   ├── defensive-security-intro.md
│   │   └── offensive-security-intro.md
│   ├── network-fundamentals/
│   │   ├── what-is-networking.md
│   │   ├── intro-to-lan.md
│   │   ├── osi-model.md
│   │   ├── packets-and-frames.md
│   │   └── extending-your-network.md
│   ├── operating-systems/
│   │   ├── linux-cli-basics.md
│   │   ├── operating-system-security.md
│   │   ├── windows-basics.md
│   │   └── windows-cli-basics.md
│   ├── web-fundamentals/
│   │   ├── how-websites-work.md
│   │   ├── dns.md
│   │   ├── http-https.md
│   │   └── web-request-flow.md
│   └── attacks-and-defenses/
│       ├── cia-triad.md
│       ├── cryptography.md
│       ├── become-a-hacker.md
│       └── become-a-defender.md
├── mkdocs.yml
├── README.md
└── .gitignore
```

---

## 🤝 Contributing

Contributions are welcome and appreciated! Whether it's fixing a typo, improving an explanation, or adding new examples — every bit helps. 🙌

### ✅ What You Can Contribute

- Fix typos or grammar mistakes
- Improve or simplify existing explanations
- Add better code examples or diagrams
- Add missing topics or subtopics
- Fix broken links or formatting issues

---

### 📋 Contribution Steps

**1. Fork the repository**

Click the **Fork** button at the top right of this page.

**2. Clone your fork**
```bash
git clone https://github.com/your-username/Cybersecurity-Basics.git
cd Cybersecurity-Basics
```

**3. Create a new branch**
```bash
git checkout -b your-branch-name
# Example: git checkout -b fix/osi-model-typo
# Example: git checkout -b add/cryptography-examples
```

**4. Set up the project locally**
```bash
python -m venv venv
source venv/bin/activate     # Mac/Linux
venv\Scripts\activate        # Windows

pip install mkdocs mkdocs-material
```

**5. Make your changes**

Edit the relevant `.md` file inside the `docs/` folder.

**6. Preview your changes**
```bash
mkdocs serve
```
Open [http://127.0.0.1:8000](http://127.0.0.1:8000) and verify everything looks correct.

**7. Commit your changes**
```bash
git add .
git commit -m "fix: corrected typo in osi-model.md"
```

**8. Push to your fork**
```bash
git push origin your-branch-name
```

**9. Open a Pull Request**

Go to the original repo on GitHub and click **"New Pull Request"**. Describe what you changed and why.

---

### 📝 Commit Message Format

Please follow this format for commit messages:

| Prefix | When to use |
|--------|-------------|
| `fix:` | Bug fix, typo correction |
| `add:` | New content or topic added |
| `update:` | Improving existing content |
| `remove:` | Removing outdated content |
| `style:` | Formatting changes only |

**Examples:**
```
fix: corrected example in osi-model.md
add: added TLS handshake diagram in http-https.md
update: improved explanation of cryptography
style: fixed code block formatting in linux-cli-basics.md
```

---

### 🗂️ Writing Style Guide

To keep the docs consistent, please follow these guidelines when writing content:

- Use **simple, beginner-friendly language**
- Always include a **code example** or diagram where applicable
- Add **expected output** after every command block
- Use admonitions for tips, warnings, and notes:
  ```
  !!! tip "Title"
      Your tip here.

  !!! warning "Title"
      Your warning here.

  !!! info "Title"
      Your info here.
  ```
- Use **tables** for comparisons (e.g., TCP vs UDP)
- Keep examples **short and focused**

---

### 🐛 Reporting Issues

Found a mistake or something that can be improved?

1. Go to the [Issues](https://github.com/your-username/Cybersecurity-Basics/issues) tab
2. Click **"New Issue"**
3. Describe the problem clearly with the page name and what needs to be fixed

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

⭐ If you find this helpful, give it a star — it keeps the motivation going!