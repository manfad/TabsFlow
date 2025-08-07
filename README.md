# TabFlow

<div align="center">

![TabFlow Logo](https://img.shields.io/badge/TabFlow-Desktop%20App-blue?style=for-the-badge&logo=vue.js&logoColor=white)

**A modern desktop note-taking application that transforms hierarchical text into structured data**

[![Vue](https://img.shields.io/badge/Vue-3.5.13-4FC08D?style=flat-square&logo=vue.js&logoColor=white)](https://vuejs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.6.2-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Tauri](https://img.shields.io/badge/Tauri-2.0-FFC131?style=flat-square&logo=tauri&logoColor=black)](https://tauri.app/)
[![License](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)](LICENSE)

[Features](#features) • [Installation](#installation) • [Usage](#usage) • [Development](#development) • [Contributing](#contributing)

</div>

## ✨ Features

- **📝 Hierarchical Text Editor**: Write structured notes with indentation-based hierarchy
- **🔧 JSON Transformation**: Automatically convert hierarchical text to structured JSON
- **🗺️ Visual Flow Mapping**: Generate interactive flow diagrams from your notes
- **💾 Multiple Export Formats**: Save as text files, JSON, or copy to clipboard
- **🎨 Modern UI**: Clean, responsive interface built with Vue 3
- **⚡ Native Performance**: Cross-platform desktop app powered by Tauri
- **🔍 Syntax Highlighting**: Monaco Editor integration for JSON editing
- **📊 Real-time Visualization**: Vue Flow diagrams that update as you type

## 🚀 Quick Start

### Prerequisites

- **Node.js** (v18 or higher)
- **Rust** (latest stable)
- **pnpm** (recommended package manager)

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/manfad/TabsFlow.git
   cd TabsFlow
   ```

2. **Install dependencies**
   ```bash
   pnpm install
   ```

3. **Run in development mode**
   ```bash
   pnpm dev
   ```

4. **Build for production**
   ```bash
   pnpm tauri build
   ```

## 📖 Usage

### Hierarchical Text Input

Write your notes using indentation to create hierarchy:

```
Project Alpha
    Planning
        Requirements gathering
        User stories
    Development
        Frontend
            Vue components
            TypeScript setup
        Backend
            API design
            Database schema
    Testing
        Unit tests
        Integration tests
```

### JSON Transformation

TabFlow automatically converts your hierarchical text into structured JSON:

```json
[
  {
    "text": "Project Alpha",
    "children": [
      {
        "text": "Planning",
        "children": [
          { "text": "Requirements gathering" },
          { "text": "User stories" }
        ]
      },
      {
        "text": "Development",
        "children": [
          {
            "text": "Frontend",
            "children": [
              { "text": "Vue components" },
              { "text": "TypeScript setup" }
            ]
          },
          {
            "text": "Backend",
            "children": [
              { "text": "API design" },
              { "text": "Database schema" }
            ]
          }
        ]
      },
      {
        "text": "Testing",
        "children": [
          { "text": "Unit tests" },
          { "text": "Integration tests" }
        ]
      }
    ]
  }
]
```

### Visual Flow Mapping

Create interactive diagrams with descriptions:

```
Project Alpha: Main development project
    Planning: Initial project setup
        Requirements gathering: Collect user needs
        User stories: Define user scenarios
    Development: Core development phase
        Frontend: User interface development
            Vue components: Reusable UI components
            TypeScript setup: Type safety configuration
        Backend: Server-side development
            API design: RESTful API structure
            Database schema: Data model design
    Testing: Quality assurance
        Unit tests: Component testing
        Integration tests: System testing
```

## 🛠️ Tech Stack

### Core Technologies
- **[Vue 3](https://vuejs.org/)** - Progressive JavaScript framework
- **[TypeScript](https://www.typescriptlang.org/)** - Type-safe JavaScript
- **[Tauri](https://tauri.app/)** - Cross-platform desktop framework
- **[Vite](https://vitejs.dev/)** - Fast build tool and dev server

### Key Dependencies
- **[@guolao/vue-monaco-editor](https://github.com/imguolao/monaco-vue)** - Monaco Editor integration for Vue
- **[@vue-flow/core](https://github.com/bcakmakoglu/vue-flow)** - Interactive flow diagrams
- **[@tauri-apps/api](https://tauri.app/v2/api/)** - Tauri desktop APIs

## 🏗️ Project Structure

```
TabsFlow/
├── src/                    # Vue frontend source
│   ├── components/        # Vue components
│   │   ├── Notepad.vue   # Text editor component
│   │   ├── JsonPad.vue   # JSON editor with Monaco
│   │   └── MapPad.vue    # Flow diagram component
│   ├── assets/           # Static assets
│   └── App.vue          # Main application component
├── src-tauri/            # Tauri backend (Rust)
├── public/              # Static files
└── package.json         # Node.js dependencies
```

## 🧪 Development

### Available Scripts

```bash
# Development
pnpm dev              # Start Vite dev server
pnpm tauri dev        # Start Tauri with hot reload

# Building
pnpm build            # Build Vue app
pnpm tauri build      # Build desktop application

# Preview
pnpm preview          # Preview production build
```

### Development Guidelines

- Use Vue 3 Composition API with `<script setup>` syntax
- Follow TypeScript best practices for type safety
- Implement proper component communication with props and emits
- Use reactive state management with `ref()` and `computed()`

## 🤝 Contributing

We welcome contributions from the community! Here's how you can help:

### Getting Started

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Make your changes following our coding standards
4. Commit your changes: `git commit -m 'Add amazing feature'`
5. Push to the branch: `git push origin feature/amazing-feature`
6. Open a Pull Request

### Development Standards

- Follow Vue 3 + TypeScript coding standards
- Write clean, documented code
- Add tests for new features
- Update documentation as needed

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

This project is built on the shoulders of amazing open source libraries:

- **[@guolao/vue-monaco-editor](https://github.com/imguolao/monaco-vue)** - Monaco Editor integration for Vue 2&3, loaded from CDN without bundling
- **[@vue-flow/core](https://github.com/bcakmakoglu/vue-flow)** - Interactive flow diagrams and node-based editors for Vue 3

Special thanks to the maintainers and contributors of these libraries for making this project possible.

## 📞 Support

- **Issues**: [GitHub Issues](https://github.com/manfad/TabsFlow/issues)
- **Discussions**: [GitHub Discussions](https://github.com/manfad/TabsFlow/discussions)
- **Documentation**: [Project Wiki](https://github.com/manfad/TabsFlow/wiki)

---

<div align="center">

Made with ❤️ by the TabFlow community

[![GitHub stars](https://img.shields.io/github/stars/manfad/TabsFlow?style=social)](https://github.com/manfad/TabsFlow)
[![GitHub forks](https://img.shields.io/github/forks/manfad/TabsFlow?style=social)](https://github.com/manfad/TabsFlow)

</div>
