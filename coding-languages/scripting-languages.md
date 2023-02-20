---
title: Ruby, PHP, Perl, and the Scripting Tradition
description: The glue languages - dynamically typed, no build step, optimized for getting something working in an afternoon. Perl made text processing a first-class idea, Ruby made everything an object you can reopen, PHP owns the request-response web, and Lua embeds anywhere. What they share and where each still wins.
date: 2023-02-20
draft: false
slug: /coding-languages/scripting-languages
tags:
  - Programming Languages
  - Web
---

"Scripting language" started as a mild insult — a language for small automation *scripts*, not *real* programs. The label stuck even as Ruby grew Rails, PHP grew Facebook, and Python grew... everything. What actually unites the family is a set of design choices aimed at developer speed over machine speed: dynamic typing, no compile step, a REPL, batteries-included standard libraries, and a forgiving attitude that lets you get something running before you fully understand the problem.

This post is the shared traits and the four languages — Perl, Ruby, PHP, Lua — where those traits pay off differently. ([Python](/citadel/coding-languages/python) has its own post; [JavaScript](/citadel/coding-languages/javascript) too.)

## What "scripting" actually means

- **Dynamic typing** — a variable holds whatever you put in it; types are checked (or coerced) at runtime. No type declarations to write, and no compiler telling you a refactor broke something until you run it.
- **Late binding / duck typing** — "if it responds to `.quack()`, it's a duck." Code depends on behaviour, not declared interfaces, so you can pass anything that works.
- **No separate build** — the interpreter reads source and runs it. Edit, save, run — the loop is seconds, and there's no `.o` files or link step.
- **REPL** — an interactive prompt to try expressions, inspect objects, and explore a library without writing a program.
- **Batteries included** — HTTP, JSON, regex, file globbing, dates, and often a package manager, all in the standard library or one install away.
- **Metaprogramming** — code that writes or modifies code at runtime: define methods dynamically, intercept missing-method calls, reopen existing classes.

The trade is real: dynamic typing catches fewer bugs at author time, interpreted execution is slower than compiled, and a large dynamically-typed codebase is harder to refactor safely — which is why every one of these languages has since grown an optional type checker.

## Perl — text is the domain

Perl (Larry Wall, 1987) was built for wrangling text and system administration, and it shows in the design: **regular expressions are syntax**, not a library call (`$line =~ s/foo/bar/g`); special variables (`$_`, `@ARGV`, `$/`) make common patterns terse; and it's genuinely good at one-liners:

```perl
perl -ne 'print if /ERROR/ && !/timeout/' server.log
```

**CPAN**, Perl's package archive (1995), was the first great one — a module for nearly everything, decades before `npm` or `pip`. Perl 5 still runs a lot of glue code, bioinformatics pipelines, and legacy web backends. Its reputation for write-only code comes from "there's more than one way to do it" taken to an extreme; disciplined Perl is readable, but the language doesn't push you there. (Perl 6 became **Raku**, a separate language.)

## Ruby — everything is an object, and you can reopen it

Ruby (Yukihiro Matsumoto, 1995) optimises for programmer happiness and consistency. *Everything* is an object with methods — `5.times { ... }`, `"hello".upcase`, `nil.to_s`. **Blocks** (closures passed to methods) make iteration and resource management clean: `File.open("x") { |f| ... }` closes the file automatically.

Its defining feature is **metaprogramming**: you can reopen any class (including built-ins) and add methods, define methods at runtime from data, and hook `method_missing` to handle calls that don't exist. This is powerful and dangerous — it's how **Rails** builds an entire ORM where `User.find_by_email_and_status(...)` just works (the method is synthesised from its name), and also how two libraries can silently break each other by patching the same class.

Rails (2004) popularised **convention over configuration** — follow the naming conventions and the framework wires everything up with no config files. It defined the shape of the "productive web framework" that Django, Laravel, and Phoenix followed.

## PHP — the web's default deployment model

PHP's model is unusual and underrated: **each request starts with a clean slate**. A web request comes in, the PHP interpreter runs your script from scratch, produces HTML, and throws away all state. No long-lived process to leak memory or hold stale state, no restart needed to deploy (just replace the files), trivially horizontally scalable. It's why PHP powered the early web and still runs an enormous share of it — WordPress, Wikipedia, and (heavily modified) Facebook.

Early PHP earned its bad reputation: inconsistent function names, `register_globals` security holes, weak typing surprises. **Modern PHP** (8.x) is a different language — typed properties and parameters, union types, enums, attributes, a JIT, `match` expressions, and a solid package manager (Composer) with a strong framework ecosystem (Laravel, Symfony). The stateless-per-request model, meanwhile, was always ahead of its time — it's essentially what serverless functions rediscovered.

## Lua — the embeddable one

Lua (1993) is tiny (the whole interpreter is a few hundred KB), fast (LuaJIT is one of the fastest dynamic-language implementations anywhere), and designed to be **embedded** in a host application written in C. The entire data model is one structure — the **table**, an associative array that serves as list, dict, object, and namespace. **Coroutines** give cooperative multitasking without threads.

It's the scripting layer inside games (World of Warcraft addons, Roblox, LÖVE), Redis (`EVAL`), Nginx (OpenResty), Neovim's config, and countless embedded devices — anywhere you want to expose scripting to users or plugin authors without shipping a heavyweight runtime.

## Shared themes, and where they bite

- **Duck typing** makes code flexible and makes "what does this actually accept?" hard to answer without running it.
- **Monkey patching / reopening classes** enables elegant DSLs and painful action-at-a-distance bugs.
- **Truthiness and coercion rules** differ per language and are a classic source of surprise (`"0"` is falsy in PHP, truthy in Ruby; `==` vs `===`).
- **DSLs** — these languages are good hosts for domain-specific mini-languages (Rails routes, RSpec, Rake, Capistrano), which read beautifully and debug poorly.
- **Retrofitted typing** — Sorbet and RBS for Ruby, Psalm/PHPStan for PHP, all bolt gradual static checks onto codebases that outgrew "just run it."

## The one idea to keep

Scripting languages trade machine performance and author-time type safety for a fast edit-run loop, dynamic typing, and heavy metaprogramming — you can build something working before you fully understand it, and reshape the language to fit the problem. Perl makes text and regexes first-class, Ruby makes every class reopenable (Rails' magic and its footguns both), PHP's start-fresh-per-request model quietly prefigured serverless, and Lua is the one you drop into a C program when you need users to be able to script it.
