# ISO 20022 Explorer

A browser-based tool for exploring the ISO 20022 financial messaging standard. Load the official eRepository XML file and navigate business areas, message definitions, and their full field structures.

## Usage

The app is distributed as a single self-contained `index.html` — no server or internet connection required. Open it in a browser and load your local copy of the ISO 20022 eRepository file (`.iso20022`).

The eRepository file is published by [ISO 20022](https://www.iso20022.org) and is not included in this repository due to its size.

## Development

```bash
npm install
npm run dev      # Start dev server at http://localhost:5173
npm run build    # Build single-file HTML bundle to dist/
npm run preview  # Preview the production build locally
npm run lint     # Run ESLint
```
