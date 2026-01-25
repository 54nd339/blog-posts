# Blog Posts

This repository contains all blog post content (markdown files and images) that can be shared across multiple projects.

## Usage

### As Git Submodule

Add this repository as a submodule to your project:

```bash
git submodule add https://github.com/54nd339/blog-posts.git content/posts
```

To update the submodule:

```bash
git submodule update --remote content/posts
```

### Clone with Submodules

When cloning a project that uses this as a submodule:

```bash
git clone --recurse-submodules <repository-url>
```

Or if already cloned:

```bash
git submodule update --init --recursive
```

## Structure

Categories are top-level directories. Larger ones are split into subcategory
folders (e.g. `dsa/` follows the NeetCode-150 patterns, `cp/` the
cp-algorithms.com sections, `physics/` its branches). A post's URL comes from its
frontmatter `slug` (`/<category>/<filename-stem>`), not its path, so the
subfolders are purely for organisation.

```
├── algorithms/            # dynamic-programming/ greedy/ divide-and-conquer/ backtracking/
├── artificial-intelligence/  # machine-learning/ deep-learning/ nlp/ search/ fuzzy-logic/ …
├── automata/
├── big-data/
├── coding-languages/
├── compiler-design/
├── computer-architecture/
├── computer-graphics/
├── computer-networks/
├── cp/                    # algebra/ data-structures/ graphs/ geometry/ string-processing/ …
├── cyber-security/
├── data-structures/
├── dbms/
├── dsa/                   # arrays-and-hashing/ two-pointers/ … bit-manipulation/ (18 patterns)
├── images/
├── interview/             # networking/ databases-storage/ security-auth/ web-apis/ …
├── maths/                 # algebra/ calculus/ geometry/ linear-algebra/ …
├── miscs/
├── operating-system/
├── parallel-computing/
├── pentesting/
├── physics/               # classical-mechanics/ electromagnetism/ quantum/ optics/ …
├── random-thoughts/
├── system-design/         # case-studies/ payments/ design-problems/
└── tech/                  # databases/ big-data/ devops/
```

## License

MIT
