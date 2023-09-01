[![License](https://img.shields.io/badge/License-BSD_3--Clause-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)

# [NVIDIA Grace CPU Benchmarking Guide][published]

**:warning: This is the guide's source code, not [the published guide][published]. :warning:**  We recommend reading the published guide: https://jlinford.gitlab-master-pages.nvidia.com/grace-cpu-benchmarking-guide/


## Working with a local copy

This guide is written with [mdBook](https://github.com/rust-lang/mdBook). To view a local copy of this guide, use `mdbook serve` and view the guide in your web browser:
```bash
cargo install mdbook
mdbook serve
```
Navigate to the URL shown in the console output, e.g.
```
2023-09-01 12:26:55 [INFO] (warp::server): listening on http://127.0.0.1:3000 
```

If you do not wish to use mdBook, this source code is readable by most humans.
You'll find everything in the [`src`](src/) directory.
Follow the paths shown in [`src/SUMMARY.md`](src/SUMMARY.md).

-----

Contact: jlinford@nvidia.com



[published]: <https://jlinford.gitlab-master-pages.nvidia.com/grace-cpu-benchmarking-guide/> "NVIDIA Grace CPU Benchmarking Guide"