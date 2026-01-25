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

```
├── algorithms/
├── artificial-intelligence/
├── automata/
├── big-data/
├── coding-languages/
├── compiler-design/
├── computer-architecture/
├── computer-graphics/
├── computer-networks/
├── cp/
├── cyber-security/
├── data-structures/
├── dbms/
├── images/
├── interview/
├── maths/
├── miscs/
├── operating-system/
├── parallel-computing/
├── pentesting/
├── physics/
├── random-thoughts/
├── system-design/
└── tech/
```

## License

MIT
