# Note App

A modern desktop note-taking application built with Tauri, Vue 3, and TypeScript.

## Features

- Cross-platform desktop application
- Built with Vue 3 and TypeScript
- Modern UI with Vite build system
- Native performance with Tauri

## Tech Stack

- **Frontend**: Vue 3 + TypeScript
- **Build Tool**: Vite
- **Desktop Framework**: Tauri (Rust)
- **Package Manager**: pnpm

## Development

### Prerequisites

- Node.js (v18 or higher)
- Rust (latest stable)
- pnpm

### Setup

1. Clone the repository:
```bash
git clone <your-repo-url>
cd note
```

2. Install dependencies:
```bash
pnpm install
```

3. Run in development mode:
```bash
pnpm dev
```

4. Build for production:
```bash
pnpm build
```

5. Run Tauri commands:
```bash
pnpm tauri dev    # Development with hot reload
pnpm tauri build  # Build desktop application
```

## Project Structure

```
note/
├── src/              # Vue frontend source
├── src-tauri/        # Tauri backend (Rust)
├── public/           # Static assets
└── package.json      # Node.js dependencies
```

## Building

To build the desktop application:

```bash
pnpm tauri build
```

This will create platform-specific installers in `src-tauri/target/release/`.

## License

MIT
