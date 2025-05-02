# LupusOS Documentation Hub

Welcome to the official documentation hub for **LupusOS**, a lightweight, community-driven Linux distribution powered by the PISI package management system. This repository contains the source files for the LupusOS developer documentation, designed to guide contributors in packaging, kernel development, and system configuration.

The documentation is built using [Sphinx](https://www.sphinx-doc.org/) and hosted on Read the Docs.

## 📚 Online Documentation

📖 View the full documentation at: [https://docs.lupusos.org](https://docs.lupusos.org)

> **Note:** Confirm the official Read the Docs URL for LupusOS documentation. Contributors should check the LupusOS website or GitHub for the correct link.

## 📁 Directory Structure

```
docs/
  └── Developer/
      ├── index.rst               # Main entry point for the developer documentation
      ├── packages/index.rst      # Guide for creating and managing PISI packages
      ├── kernel/index.rst        # Instructions for kernel development and module creation
      ├── contributing.rst        # Contribution guidelines for developers
      ├── installer/index.rst     # Documentation for installer development
      ├── modules/index.rst       # Reference for the PISI Actions API
├── .readthedocs.yaml           # Configuration file for Read the Docs builds
├── requirements.txt            # Python dependencies required to build the documentation
└── README.md                   # This file, providing an overview of the repository
```

## 🛠️ Building Locally

To build and preview the documentation on your local machine, follow these steps:

### Prerequisites

- Python 3.8 or higher
- `pip` for installing Python dependencies
- Optional: `make` for Unix-like systems (Windows users can use `sphinx-build` directly)

### Steps

1. Clone the repository:

   ```bash
   git clone https://github.com/LupusOS/documentation-hub.git
   cd documentation-hub
   ```

2. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

3. Build the HTML documentation:

   ```bash
   cd docs
   make html
   ```

   On Windows, use:

   ```bash
   sphinx-build -b html . _build/html
   ```

4. Open the generated documentation:

   ```bash
   open _build/html/index.html  # On macOS
   xdg-open _build/html/index.html  # On Linux
   start _build/html/index.html  # On Windows
   ```

The documentation will be available in the `_build/html/` directory.

## 🤝 Contributions

We welcome contributions to improve the LupusOS documentation! Whether you're fixing typos, adding new sections, or updating guides, your efforts help the community.

To get started, see the [contribution guidelines](docs/Developer/contributing.rst) for detailed instructions on submitting pull requests and collaborating with the LupusOS community.

> **Note:** Verify the official LupusOS community channels (e.g., forums, IRC, mailing list). Contributors should check the LupusOS website or GitHub for up-to-date contact information.

## 📜 License

This documentation is released under the MIT License. See the [LICENSE](LICENSE) file for details.
