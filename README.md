# Pyssla Creator

A modern, web-based tool for designing Pyssla (perler bead) patterns. Built with Vue 3 and Vite.

## Features

- **Interactive Grid**: Customizable grid size (width/height) to match your pegboard.
- **Drawing Tools**: Click or drag to paint beads.
- **Color Palette**: 
  - curated list of vibrant colors.
  - Add custom colors using the color picker.
- **History**: Undo (Ctrl+Z) and Redo (Ctrl+Y) support.
- **Persistence**: Save your designs to JSON and load them back later.
- **Modern UI**: Sleek dark mode interface.

## Getting Started

### Prerequisites

- Node.js (v14+ recommended)
- npm

### Installation

1. Clone the repository (or navigate to the project directory).
2. Install dependencies:
   ```bash
   npm install
   ```

### Running the App

Start the development server:

```bash
npm run dev
```

Open [http://localhost:5173](http://localhost:5173) in your browser.

## File Format

The application uses a custom JSON format for saving patterns:

```json
{
  "version": 1,
  "width": 20,
  "height": 20,
  "palette": ["#000000", "#ffffff", ...],
  "grid": [
    ["#000000", "", ...],
    ...
  ]
}
```

## License

MIT
