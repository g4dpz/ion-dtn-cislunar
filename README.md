# ION-DTN Cislunar Demonstration

Repository: https://github.com/g4dpz/ion-dtn-cislunar

A phased demonstration of Delay-Tolerant Networking using NASA JPL's ION implementation, progressing from a terrestrial testbed through a CubeSat to a cislunar payload. Led by AMSAT-UK and AMSAT-DL for the worldwide amateur radio and academic communities.

## Project Documents

| Document | Source | PDF |
|----------|--------|-----|
| Executive Summary | `docs/executive-summary.md` | `docs/executive-summary.pdf` |
| Requirements | `docs/requirements.md` | — |
| Design | `docs/design.md` | — |
| Tasks (overview) | `docs/tasks.md` | — |

## Prerequisites

PDF generation requires:

- [pandoc](https://pandoc.org/) (markdown to PDF conversion)
- [XeLaTeX](https://tug.org/xetex/) (PDF engine with Unicode support)
- Menlo font (bundled with macOS; substitute any monospace font on other platforms)

On macOS with Homebrew:

```bash
brew install pandoc
brew install --cask mactex
```

On Debian/Ubuntu:

```bash
sudo apt install pandoc texlive-xetex
```

## Generating PDFs

### Quick build (manual version)

```bash
# Requirements
pandoc docs/requirements.md \
  -o docs/requirements.pdf \
  --pdf-engine=xelatex \
  -V geometry:margin=2.5cm -V fontsize=11pt -V documentclass=article \
  --toc -V toc-title="Table of Contents" \
  -V title="ION-DTN Cislunar Demonstration — Requirements Document" \
  -V date="$(date +%B\ %Y)" -V monofont="Menlo" \
  --resource-path=docs

# Design
pandoc docs/design.md \
  -o docs/design.pdf \
  --pdf-engine=xelatex \
  -V geometry:margin=2.5cm -V fontsize=11pt -V documentclass=article \
  --toc -V toc-title="Table of Contents" \
  -V title="ION-DTN Cislunar Demonstration — Design Document" \
  -V date="$(date +%B\ %Y)" -V monofont="Menlo" \
  --resource-path=docs

# Executive Summary
pandoc docs/executive-summary.md \
  -o docs/executive-summary.pdf \
  --pdf-engine=xelatex \
  -V geometry:margin=2.5cm -V fontsize=11pt -V documentclass=article \
  -V title="ION-DTN Cislunar Demonstration" \
  -V subtitle="Executive Summary" \
  -V date="$(date +%B\ %Y)" -V monofont="Menlo"
```

### Versioned build (from git tag)

The document version can be derived automatically from the git tag. This ensures the PDF version always matches the repository release.

```bash
# Get version from git
VERSION=$(git describe --tags --always 2>/dev/null || echo "untagged")
DATE=$(date +"%B %Y")

# Build all PDFs with version injected
for doc in requirements design executive-summary; do
  pandoc docs/${doc}.md \
    -o docs/${doc}.pdf \
    --pdf-engine=xelatex \
    -V geometry:margin=2.5cm -V fontsize=11pt -V documentclass=article \
    --toc -V toc-title="Table of Contents" \
    -V title="ION-DTN Cislunar Demonstration" \
    -V subtitle="Version ${VERSION}" \
    -V date="${DATE}" -V monofont="Menlo" \
    --resource-path=docs
done
```

### Tagging a release

```bash
# Tag a new version
git tag -a v0.1.0 -m "Initial draft of requirements, design, and executive summary"
git push origin v0.1.0

# Rebuild PDFs — version will now show as v0.1.0
VERSION=$(git describe --tags --always)
# ... run pandoc commands above
```

### How versioning works

- The `Version` field in each document's header table is maintained manually in the markdown source for the change history record
- The PDF title page version is injected at build time from `git describe --tags --always`
- `git describe` output format: `v0.1.0` (on a tag), `v0.1.0-3-gabcdef` (3 commits after tag), or a short commit hash if no tags exist
- The change history table in each document is maintained manually as a human-readable changelog — it records what changed in each version, not just the version number

## Repository Structure

```
.
├── README.md                          # This file
└── docs/
    ├── executive-summary.md           # Executive summary for agencies/funders
    ├── executive-summary.pdf          # Generated PDF
    ├── requirements.md                # Requirements document (34 requirements)
    ├── design.md                      # Design document (65 correctness properties)
    ├── tasks.md                       # Implementation plan overview and task index
    ├── figures/                       # Generated diagram images
    │   ├── figure1_phases.png
    │   ├── figure2_topology.png
    │   └── figure3_protocol_stack.png
    ├── master/                        # Cross-cutting: CI/CD, integration, optional
    │   ├── requirements.md
    │   ├── design.md
    │   ├── executive-summary.md
    │   └── tasks.md
    ├── phase1-terrestrial/            # Phase 1: Terrestrial testbed
    │   ├── requirements.md
    │   ├── design.md
    │   └── tasks.md
    ├── phase2-cubesat/                # Phase 2: CubeSat
    │   ├── requirements.md
    │   ├── design.md
    │   └── tasks.md
    └── phase3-cislunar/               # Phase 3: Cislunar payload
        ├── requirements.md
        ├── design.md
        └── tasks.md
```

## ION-DTN Reference Implementation

The project wraps NASA JPL's ION-DTN, symlinked at `ION-DTN/`. To set up the symlink:

```bash
ln -s /path/to/your/ION-DTN ./ION-DTN
```

## Licence

All project software, documentation, and hardware designs are released under the Apache License, Version 2.0.
