# [NVIDIA Grace CPU Benchmarking Guide][1]

**:warning: This is the guide's source code, not [the published guide][1]. :warning:**  It's much easier to read and navigate the published guide: https://jlinford.gitlab-master-pages.nvidia.com/grace-cpu-benchmarking-guide/


## Working with a local copy

This guide is written with [mdBook](https://github.com/rust-lang/mdBook). To view a local copy of this guide, use `mdbook serve` and view the guide in your web browser:
```
cargo install mdbook
mdbook serve
```
Navigate to the URL shown in the console output, e.g.
```
2023-09-01 12:26:55 [INFO] (warp::server): listening on http://127.0.0.1:3000 
```

If you do not wish to use mdBook, the code in the [`src`](src/) directory is readable by most humans.
Follow the paths shown in [`src/SUMMARY.md`](src/SUMMARY.md).

## License

Unless otherwise indicated, this work is licensed under
[The 3-Clause BSD License](https://opensource.org/license/bsd-3-clause/).  Individual examples or attached source code may be under a different license.  Check the related README or LICENSE files.


-----

Copyright (c) 2023 NVIDIA CORPORATION & AFFILIATES. All rights reserved.



[1]: <https://jlinford.gitlab-master-pages.nvidia.com/grace-cpu-benchmarking-guide/> "NVIDIA Grace CPU Benchmarking Guide"