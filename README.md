# RADOLAN Guide

## Overview

This repository contains a set of notebooks and utilities for working with RADOLAN data. It was part of [wradlib-notebooks](https://github.com/wradlib/wradlib-notebooks/) and was transferred into its own repository when reorganizing and restructuring the wradlib documentation. 

The documentation and notebooks of the *RADOLAN Guide* are now available at https://docs.wradlib.org/projects/radolan-guide.

RADOLAN is abbreviated from the german **RA**dar-**O**n**L**ine-**AN**eichung, which means Radar-Online-Adjustment.

Using it's [network of 17 weather radar](https://www.dwd.de/SharedDocs/broschueren/DE/presse/wetterradar_pdf.pdf?__blob=publicationFile&v=5) The Deutscher Wetterdienst provides [many products](https://www.dwd.de/DE/leistungen/radolan/produktuebersicht/radolan_produktuebersicht_pdf.pdf?__blob=publicationFile&v=6) for high resolution precipitation analysis and forecast. A comprehensive product list can be found in the [Product](notebooks/product) chapter .

The formats of the composite products are described in the [Format](notebooks/format) chapter. They consist of a binary format with an ASCII header and an ASCII-only format. All composites are available in [Polar Stereographic Projection](notebooks/grid.md#projection) which will be discussed in the chapter [Grid](notebooks/grid).

This notebook tutorial was prepared with material from the [DWD RADOLAN/RADVOR-OP Kompositformat](https://www.dwd.de/DE/leistungen/radolan/radolan_info/radolan_radvor_op_komposit_format_pdf.pdf?__blob=publicationFile&v=5).
Special thanks go to Elmar Weigl, The Deutscher Wetterdienst, for providing the extensive set of example data and his valuable information about the RADOLAN products.

## Installation

```bash
git clone https://github.com/wradlib/radolan-guide.git
cd radolan-guide
sphinx-build -b html . _build/html
```
Then open `_build/html/index.html` in your browser.

## License

- **Source code**: [MIT LICENSE](LICENSE-MIT) - Copyright (c) 2025 wradlib developers
- **Notebooks and documentation**: [CC BY-SA 4.0](LICENSE-CC-BY-SA-4.0) - Copyright (c) 2025 wradlib developers
- **Data**: [CC BY 4.0](data/ATTRIBUTION.md) - The Deutscher Wetterdienst (DWD) RADOLAN dataset
