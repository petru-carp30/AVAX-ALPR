# Third-Party Notices

This document records third-party datasets, libraries, and other external assets used or selected for use by AVAX ALPR where attribution or license preservation is required.

## Romanian (European Union) License Plate Dataset

**Upstream project:** `RobertLucian/license-plate-dataset`  
**Source:** https://github.com/RobertLucian/license-plate-dataset  
**Dataset description:** Romanian / European Union license-plate images with Pascal VOC annotations. The upstream README states that the dataset contains 534 images, with an 80% training / 20% validation organization and both daytime and nighttime images.  
**License:** MIT License  
**Copyright:** Copyright (c) 2020 Robert Lucian Chiriac

### AVAX ALPR usage status

Selected for planned use in `AI-DATA-WP-001 — License Plate Detector Dataset Acquisition & Annotation Foundation`.

The dataset must not be represented as an AVAX-created dataset. Any local copy, converted annotation set, derived split, or redistributed substantial portion based on this upstream dataset must retain the upstream attribution and license notice required by the MIT License.

### License obligations to preserve

The MIT License permits use, copying, modification, merging, publishing, distribution, sublicensing, and sale, subject to the condition that the copyright notice and permission notice are included in all copies or substantial portions of the licensed material.

For AVAX ALPR this means:

- keep the upstream `LICENSE` file with the locally acquired raw dataset;
- preserve the copyright notice `Copyright (c) 2020 Robert Lucian Chiriac`;
- preserve the MIT permission notice when the dataset or a substantial portion is redistributed;
- keep this source/attribution record in project documentation;
- retain attribution for converted annotations, derived train/validation/test splits, or other substantial dataset derivatives;
- do not imply that the original author endorses AVAX ALPR.

### Upstream MIT License notice

```text
MIT License

Copyright (c) 2020 Robert Lucian Chiriac

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

### Provenance / legal review note

The upstream repository publishes an MIT `LICENSE` at repository level and describes the repository as a Romanian / EU license-plate dataset. The upstream README does not separately document the provenance or image-level rights for every photograph.

Therefore AVAX ALPR should preserve the repository-level MIT attribution as required, while also keeping this provenance note. Before any external redistribution of the raw images or a commercial release that bundles the dataset itself, image-level provenance should be reviewed if additional certainty is required.

The trained model may be distributed separately from the raw dataset subject to the licenses of the model architecture/runtime and any additional datasets used. Those licenses must be recorded independently in this file before release.
